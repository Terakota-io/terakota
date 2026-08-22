# Verify your download

Every terakota release is checksummed and signed. Verifying before you run the
binaries confirms the bytes you hold are the exact bytes the release pipeline
produced. (Verification is about distribution integrity — it is not a claim that
the software is free of defects.)

Download these alongside your archive from the [Releases page](../../releases):

- `SHA256SUMS` — checksums for every artifact in the release
- `<your-archive>.sig` — the cosign signature for your archive
- `cosign.pub` — the public key the release is signed with

## 1. Check the SHA-256 checksum

From the directory holding your archive and `SHA256SUMS`:

    sha256sum --check --ignore-missing SHA256SUMS

`--ignore-missing` lets you verify just the file you downloaded without pulling
every artifact. You want to see `OK` for your archive.

## 2. Verify the cosign signature

terakota releases are signed with a **key** (not keyless), and the public key
ships with the release as `cosign.pub`:

    cosign verify-blob --key cosign.pub --insecure-ignore-tlog=true \
      --signature terakota_v1.7.0_linux_amd64.tar.gz.sig \
      terakota_v1.7.0_linux_amd64.tar.gz

`--insecure-ignore-tlog=true` is correct here: this release does **not** use a
transparency log. The flag name is cosign's, not a warning about your download.
A `Verified OK` means the signature matches `cosign.pub`.

You can verify the `SHA256SUMS` file itself the same way (`SHA256SUMS.sig`), which
is a convenient way to trust the whole checksum list at once, then let step 1
cover each archive.

### About the trust model (read this)

Signing is **key-based**, and the public key is distributed alongside the release.
That means trust is established on first download: you are trusting that the
`cosign.pub` you fetched with the release is the real one. There is no
transparency log to cross-check against in v1. If the signing key is ever rotated
— routinely or in an emergency — that is announced through the process in
[SECURITY.md](../SECURITY.md), including an out-of-band path if the key itself is
ever compromised. A move to keyless signing is a future item.

### Verifying a `.deb` or `.rpm`

The packages are checksummed and cosign-signed exactly like the archives — the
same two steps apply, with the package filename in place of the archive:

    sha256sum --check --ignore-missing SHA256SUMS
    cosign verify-blob --key cosign.pub --insecure-ignore-tlog=true \
      --signature terakota_v1.7.0_linux_amd64.deb.sig \
      terakota_v1.7.0_linux_amd64.deb

The packages are **not** GPG-signed for `apt`/`dnf`; the cosign signature above
is the signature. Note also that a package is a repackaging of the matching
archive, never a separate build: the binaries inside it are byte-identical to the
ones in `terakota_<tag>_linux_<arch>.tar.gz`, and the release pipeline asserts
that by digest before publishing. You can check it yourself:

    dpkg-deb --fsys-tarfile terakota_v1.7.0_linux_amd64.deb | tar -xO ./usr/bin/terakota | sha256sum
    tar -xzOf terakota_v1.7.0_linux_amd64.tar.gz terakota | sha256sum

## 3. SBOM and provenance (what the extra files are)

Each release also carries, per target:

- **SBOM** — `*.spdx.json`, a Software Bill of Materials in SPDX format listing
  the components built into each binary. Useful for your own vulnerability and
  license scanning. There is one SBOM **per archive**, and it covers the packages
  too: a package's binaries are the same bytes as the matching archive's, so
  `terakota_<tag>_linux_<arch>.tar.gz.spdx.json` is the bill of materials for
  `terakota_<tag>_linux_<arch>.deb` and `.rpm` as well.
- **SLSA provenance** — a signed attestation binding each artifact to the source
  commit and build that produced it.

These are supplementary evidence about how the artifacts were built. Steps 1 and 2
are what you need to trust a download; the SBOM and provenance are there when you
want to inspect further.

## Verifying receipt chains is a different thing

The steps above verify your **download**. Checking a receipt chain that terakota
produced on your machine is a separate operation with the bundled
`verify-receipts` binary:

    terakota export --company mybooks --out chain.jsonl
    verify-receipts chain.jsonl

That checks the internal integrity of a chain you custodied yourself. See
[faq.md](faq.md) for what receipts do and do not prove.

### The evidence pack (from v1.6.0)

`terakota evidence --company mybooks --out pack.zip` composes the same chain into a
pack you can hand to someone: `receipts.jsonl`, a scannable `receipts.csv`, a
printable `timeline.html`, a `manifest.json`, and a `VERIFY.md` walkthrough.
Composition is verify-gated — a chain that fails verification writes nothing — and
the pack re-checks with the same standalone binary, no terakota process running:

    verify-receipts receipts.jsonl

Exit `0` is verified. Exit `4` is verified with an incomplete tail: everything
present checks out, and some execution was intended with no outcome recorded after
it — a crash or a killed process leaves exactly that. `1` is a real failure and `2`
means the file could not be read.

If the chain carries **connection records**, a bare run holds each record's own
columns to the connect broker's recorded statement beside them. Pinning the
broker's public key additionally re-checks the broker's signature over those bytes:

    verify-receipts -connect-broker-key 3nB5pydGk77NBp1m4EIDnFRp3Fff6Ig_IyCUwsxvZcc receipts.jsonl

That key is public by design; the authoritative copy is pinned in the v1.6.0
release notes, and taking it from there rather than from the pack is the point — a
key that travelled inside the file it authenticates would prove nothing.

**From v1.7.0**, when the chain carries a reconcile, the pack carries it too: the link pairs, the review
reclass records, the matcher-run completion records and the reconciliation verdicts, each
rendered in `timeline.html` in plain English beside the reads it stands on.
`verify-receipts` grades those records **and the ordering contract between them** — a
link result has to follow its own intent, a supersede cannot precede the link it sets
aside, and a link chained after its run's completion record is out of order.

The manifest then gains a `correlation` block: the verdict's window, the classes that
verdict declared it could not reach, and its lineage — the `link_set_hash`, the
derivation/confirmation mix of the links it stood on, and the
`correlation_config_version` those links were evaluated under. It renders **one** verdict,
the newest that stood on a link set, and `earlier_records_exist` says whether others sit
behind it on the chain.

Every pack carrying link material also carries the disclosure that rides every surface
moving chain content off the machine: *link records carry the correlation key material
that formed them — for exact-dimension links that includes the entry's date, amount and
account set.* Read [reconcile.md](reconcile.md) before you hand one to anybody.

**What a pass establishes.** The evidence class is **artifact integrity**. The
records are hash-linked, so a partial change — one edited row, a truncated file, a
corrupted byte — breaks the linkage and surfaces in every check above. A wholesale
replacement does not: re-hash a chain forward, rebuild the manifest around it, and
the result is internally consistent. The remedy sits outside the pack and costs one
line — write down the `chain_head` from `manifest.json` when you receive a pack, and
a pack produced later with a different head is a different pack. The pack's own
`VERIFY.md` carries the full statement, including the by-hand procedure with stdlib
tools; read it before you rely on the pack. For a reconcile verdict in the pack, a pass
establishes that the verdict is derived from the chained reads and the link set it names
— not that the books are right.
