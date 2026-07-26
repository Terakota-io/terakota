# Install

There is no account to sign up for and nothing phones home. Pick a channel:

**Homebrew (macOS, Linux)**

    brew install terakota-io/tap/terakota

**Install script (macOS, Linux)**

    curl -fsSL https://terakota.io/install.sh | sh

Installs to `~/.local/bin`. The script downloads the official release archive,
checks it against the release's published `SHA256SUMS`, and refuses to install on
a mismatch. It is served as plain text — read it before you pipe it to a shell.

Two settings, and they go on the **`sh`** side of the pipe — putting them before
`curl` sets them for `curl` instead, which does nothing:

    # pin a release instead of taking the latest
    curl -fsSL https://terakota.io/install.sh | TERAKOTA_VERSION=v1.0.0 sh

    # install system-wide (needs write access to /usr/local)
    curl -fsSL https://terakota.io/install.sh | sudo PREFIX=/usr/local sh

**Windows, or any manual install:** follow [Download](#1-download) below.

Both channels install the same two binaries from the same signed archives, plus
`EULA.md` and `THIRD_PARTY_NOTICES`. Neither is a substitute for verifying the
release yourself — see **[verify.md](verify.md)**.

## Manual install

Every release carries archives for Linux, macOS, and Windows on both `amd64` and
`arm64`. Each archive contains:

- `terakota` — the CLI and MCP server
- `verify-receipts` — the standalone offline receipt-chain verifier
- `THIRD_PARTY_NOTICES` — open-source license notices for embedded components
- `EULA.md` — the license for the binaries

Archive names follow `terakota_<tag>_<os>_<arch>` — `<tag>` is the release tag
verbatim, including the leading `v`:

- Linux / macOS: `.tar.gz` (e.g. `terakota_v1.0.0_linux_amd64.tar.gz`,
  `terakota_v1.0.0_darwin_arm64.tar.gz`)
- Windows: `.zip` (e.g. `terakota_v1.0.0_windows_amd64.zip`)

> **QuickBooks is sandbox-only in v1.** QuickBooks Online connections in this
> release run against Intuit **sandbox** companies only. Production QuickBooks
> arrives in a later release. AppFolio reads are unaffected. Everything is
> read-only by construction.

## 1. Download

Grab the archive for your OS and CPU from the [Releases page](../../releases),
along with `SHA256SUMS`, the matching `.sig` signature, and `cosign.pub`.

Not sure which `arch`? On Linux/macOS run `uname -m` — `x86_64` means `amd64`,
`aarch64`/`arm64` means `arm64`. On Apple Silicon (M-series) use `darwin_arm64`.

## 2. Verify (do this before extracting)

Every archive has a SHA-256 checksum and a cosign signature, and each release
carries SBOMs and build provenance. Verify the bytes you hold before you run
them — see **[verify.md](verify.md)** for the exact commands.

## 3. Extract and put on PATH

Linux / macOS — substitute the archive you downloaded (`darwin_arm64` on Apple
Silicon, `darwin_amd64` on Intel Macs, `linux_amd64`/`linux_arm64` on Linux):

    tar -xzf terakota_v1.0.0_darwin_arm64.tar.gz
    install -m 0755 terakota verify-receipts /usr/local/bin/   # or any dir on your PATH

Windows (PowerShell): extract the `.zip` and move `terakota.exe` and
`verify-receipts.exe` into a folder on your `PATH` (e.g. a `bin` folder you add
to the user `Path` environment variable).

## Unsigned-binary warnings (expected in v1)

The binaries are cosign-signed but **not yet OS code-signed**. Your OS may warn
that the publisher is unverified. This is expected at v1; OS code-signing arrives
in a later release. Verify the download per [verify.md](verify.md), then:

- **macOS (Gatekeeper).** The first run may be blocked as "from an unidentified
  developer." Clear the quarantine attribute:

      xattr -d com.apple.quarantine ./terakota
      xattr -d com.apple.quarantine ./verify-receipts

  Or open each binary once via right-click → **Open** in Finder and confirm.

- **Windows (SmartScreen).** You may see "Windows protected your PC." Click
  **More info → Run anyway**.

Only do this after you have verified the download.

## First run

    terakota company add --company mybooks --base-url https://<yourdomain>.appfolio.com/api/v2
    terakota credentials set --company mybooks     # no-echo prompts; sealed in an encrypted local keystore
    terakota qbo connect --company mybooks         # optional: QuickBooks OAuth against an Intuit sandbox company

The first run prints a one-time disclosure notice. You can re-show it any time
with `terakota about`, and print the third-party license notices with
`terakota licenses`.

Then any read tool works, e.g.:

    terakota appfolio_bills_list --company mybooks --updated-from 2026-07-01

## MCP mode

To use terakota as an MCP server, configure your MCP host to run
`terakota mcp --company mybooks`. See the host's documentation for where server
commands are configured. Pass the keystore passphrase via
`TERAKOTA_KEYSTORE_PASSPHRASE_FD` (an open file descriptor — never a plain
environment value).
