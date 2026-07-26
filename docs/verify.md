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
      --signature terakota_v1.0.0_linux_amd64.tar.gz.sig \
      terakota_v1.0.0_linux_amd64.tar.gz

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
      --signature terakota_v1.1.0_linux_amd64.deb.sig \
      terakota_v1.1.0_linux_amd64.deb

The packages are **not** GPG-signed for `apt`/`dnf`; the cosign signature above
is the signature. Note also that a package is a repackaging of the matching
archive, never a separate build: the binaries inside it are byte-identical to the
ones in `terakota_<tag>_linux_<arch>.tar.gz`, and the release pipeline asserts
that by digest before publishing. You can check it yourself:

    dpkg-deb --fsys-tarfile terakota_v1.1.0_linux_amd64.deb | tar -xO ./usr/bin/terakota | sha256sum
    tar -xzOf terakota_v1.1.0_linux_amd64.tar.gz terakota | sha256sum

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
want to inspect or audit further.

## Verifying receipt chains is a different thing

The steps above verify your **download**. Checking a receipt chain that terakota
produced on your machine is a separate operation with the bundled
`verify-receipts` binary:

    terakota export --company mybooks --out chain.jsonl
    verify-receipts chain.jsonl

That checks the internal integrity of a chain you custodied yourself. See
[faq.md](faq.md) for what receipts do and do not prove.
