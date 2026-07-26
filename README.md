# terakota

**terakota** is a free, local-first command-line tool and MCP server that reads the
AppFolio and QuickBooks Online accounts you already run — using your own
credentials, on your own machine — and records every read as a verifiable
integrity receipt.

It is read-only by construction, sends nothing to us, and needs no account with
us. It ships with `verify-receipts`, a standalone tool that checks receipt chains
offline.

## What it does

- **Reads your systems with your credentials.** terakota connects to AppFolio and
  QuickBooks Online using credentials you supply, against accounts you are
  authorized to access. Its only network connections are to those systems.
- **Read-only by construction.** The binaries contain no code paths that write to
  the connected systems — a structural property of the shipped client, asserted by
  automated checks at build time.
- **Every read is receipted.** For each read, terakota records a declared-intent
  entry before it acts and an executed-result entry after, on a hash chain stored
  on your machine. These are **integrity records of what terakota executed**,
  verifiable offline with `verify-receipts`. Their limits, stated plainly: they
  are **not** a log of an AI agent's full activity or reasoning, and — because the
  chain lives under your control — they are **not** tamper-evident against whoever
  controls the machine. Treat them as integrity evidence for a chain you
  custodied, not as third-party attestation.
- **Zero phone-home.** No telemetry, no analytics, no crash reporting, no network
  calls to any service we operate. We cannot see your credentials, queries,
  results, or receipts.

> **QuickBooks is sandbox-only in v1.** QuickBooks Online connections in this
> release run against Intuit **sandbox** companies only; production QuickBooks
> arrives in a later release. AppFolio reads are unaffected.

We are not affiliated with, endorsed by, or sponsored by AppFolio, Inc. or Intuit
Inc.; their services are governed by your agreements with them.

## Get started

On macOS or Linux:

    brew install terakota-io/tap/terakota
    # or
    curl -fsSL https://terakota.io/install.sh | sh

On Windows, download the signed `.zip` from
[Releases](../../releases). Then:

- **Install — all channels and the manual path:** [docs/install.md](docs/install.md)
- **Verify your download:** [docs/verify.md](docs/verify.md)
- **FAQ — what receipts do and do not prove:** [docs/faq.md](docs/faq.md)

## What ships here

Signed release artifacts only — the `terakota` binary and the `verify-receipts`
verifier for Linux, macOS, and Windows (`amd64` + `arm64`). Each release carries
SHA-256 checksums, cosign signatures, SPDX SBOMs, and SLSA build provenance. The
binaries are cosign-signed but not yet OS code-signed at v1 (see
[docs/install.md](docs/install.md) for the expected Gatekeeper/SmartScreen
notices). Source code is not published here.

## Legal and support

- **License for the binaries:** [docs/legal/EULA.md](docs/legal/EULA.md) — free to
  use, unmodified. Third-party notices ship in each archive and via
  `terakota licenses`.
- **Privacy:** [docs/legal/PRIVACY.md](docs/legal/PRIVACY.md)
- **Security advisories & support policy:** [SECURITY.md](SECURITY.md) — how we
  disclose problems, support windows, and how to report a vulnerability. Signed
  advisories publish under [advisories/](advisories/); watch this repository to be
  notified.
