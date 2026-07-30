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
    curl -fsSL https://terakota.io/install.sh | TERAKOTA_VERSION=v1.3.1 sh

    # install system-wide (needs write access to /usr/local)
    curl -fsSL https://terakota.io/install.sh | sudo PREFIX=/usr/local sh

**Linux packages (`.deb` / `.rpm`, since v1.1.0)**

    sudo dpkg -i terakota_v1.3.1_linux_amd64.deb
    sudo rpm -i  terakota_v1.3.1_linux_amd64.rpm

Download the package for your CPU from the [Releases page](../../releases).
Both binaries install to `/usr/bin`, and `EULA.md` + `THIRD_PARTY_NOTICES` to
`/usr/share/doc/terakota/`. The packages declare **no dependencies** — the
binaries are static.

Two things worth knowing:

- If you previously installed to `/usr/local/bin` by hand, that copy **shadows**
  the packaged one on most distributions (`/usr/local/bin` comes first on
  `PATH`). Remove it, or run `hash -r` and check `command -v terakota`.
- The RPM is **not GPG-signed**, so `dnf`/`yum` will warn or refuse under a
  strict `gpgcheck` policy. Verify the download per
  [verify.md](verify.md) and install the file directly with `rpm -i`, or pass
  `--nogpgcheck` if you have judged that acceptable.

**Windows, or any manual install:** follow [Download](#1-download) below.

Every channel installs the same two binaries from the same signed archives, plus
`EULA.md` and `THIRD_PARTY_NOTICES`. None of them is a substitute for verifying
the release yourself — see **[verify.md](verify.md)**.

## Where your credentials live

Since v1.2.0 the default store is your **OS keychain** — macOS Keychain, Windows
Credential Manager, or a Linux Secret Service. The operating system owns unlocking, so
tool calls and the MCP server start without a passphrase prompt.

If no keychain is reachable, terakota **fails closed** rather than quietly storing your
credentials somewhere less protected. The encrypted-file keystore is still available, as
an explicit choice:

    terakota keystore status                     # which backend, and why the warning
    terakota keystore use-file-fallback --yes    # opt in (warns on every command)
    terakota keystore migrate --to keychain      # move existing entries

Nothing moves between backends on its own. `migrate` is the only command that copies, and
it never deletes the source — remove the old copy yourself once you have confirmed the new
backend works.

**Upgrading from v1.0.0 or v1.1.0:** your credentials are in the encrypted file and stay
there. Run `terakota keystore migrate --to keychain` to move them, or
`terakota keystore use-file-fallback --yes` to keep using the file. Until you do one of
those, commands fail closed on a machine without a keychain.

## Use terakota as an MCP extension

Since v1.2.0 each release carries `.mcpb` bundles — one per platform and CPU, because the
bundle format has no architecture dimension. Download the one matching your machine
(`terakota_<tag>_darwin_arm64.mcpb` on Apple Silicon).

**1. Do all of this on the machine that runs the host.** Credentials live in that
machine's OS keychain, so a setup you did in WSL2 or on a Mac does not carry over to
Claude Desktop on Windows. Install the CLI there and run the two commands below there.

**2. Create the company and store its credentials — before you install the bundle.**

    terakota company add --company mybooks --base-url https://api.appfolio.com/api/v0
    terakota credentials set --company mybooks
    # extension's "Company id" field:  mybooks   <- the SAME string

`<yourdomain>.appfolio.com` is your web portal / Reports API address. The Database API
terakota speaks lives at `api.appfolio.com/api/v0` for every customer.

The extension's **Company id** field is a pointer to the company you just created — it
creates nothing and stores nothing. It has to be that same string, exactly.

**3. Install the bundle from Claude Desktop's settings.** There is no `.mcpb` file
association, so double-clicking one gets you the "select an app to open this file" picker
listing Notepad and whatever else — expected, not a defect. Open Claude Desktop →
**Settings** → **Extensions**, and install (or drag in) the `.mcpb` there.

Four things to expect after that:

- **A red "access to everything / not verified by Anthropic" banner.** Claude Desktop
  shows it for any sideloaded extension; it is not about terakota specifically. Expected
  at v1, same as the Gatekeeper/SmartScreen notices below.
- **The extension exists only in chats that run on this computer.** A chat running in the
  cloud shows no error at all — Claude simply says it cannot find terakota. If Claude
  says terakota does not exist, the chat is running in the cloud: start a new one set to
  run on this computer.
- **The CLI and the extension cannot share a company.** While the extension (or any MCP
  host) is connected, CLI commands on that same company fail with `chain is already open
  by another process`. One writer per chain, by design — close the chat or turn the
  extension off first.
- **Tools arrive set to "Needs approval".** The first call to one prompts you before it
  runs.

## Manual install

Every release carries archives for Linux, macOS, and Windows on both `amd64` and
`arm64`. Each archive contains:

- `terakota` — the CLI and MCP server
- `verify-receipts` — the standalone offline receipt-chain verifier
- `THIRD_PARTY_NOTICES` — open-source license notices for embedded components
- `EULA.md` — the license for the binaries

Archive names follow `terakota_<tag>_<os>_<arch>` — `<tag>` is the release tag
verbatim, including the leading `v`:

- Linux / macOS: `.tar.gz` (e.g. `terakota_v1.3.1_linux_amd64.tar.gz`,
  `terakota_v1.3.1_darwin_arm64.tar.gz`)
- Windows: `.zip` (e.g. `terakota_v1.3.1_windows_amd64.zip`)

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

    tar -xzf terakota_v1.3.1_darwin_arm64.tar.gz
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

If you already set up a company for the
[MCP extension](#use-terakota-as-an-mcp-extension), it is the same company — skip
to the read below.

    terakota company add --company mybooks --base-url https://api.appfolio.com/api/v0
    terakota credentials set --company mybooks     # no-echo prompts; sealed in your OS keychain
    terakota qbo connect --company mybooks         # optional: QuickBooks OAuth against an Intuit sandbox company

`<yourdomain>.appfolio.com` is your web portal / Reports API address. The Database API
terakota speaks lives at `api.appfolio.com/api/v0` for every customer.

The first run prints a one-time disclosure notice. You can re-show it any time
with `terakota about`, and print the third-party license notices with
`terakota licenses`.

Then any read tool works, e.g.:

    terakota appfolio_bills_list --company mybooks --updated-from 2026-07-01T00:00:00Z

Time bounds are RFC3339.

## MCP mode

To use terakota as an MCP server, configure your MCP host to run
`terakota mcp --company mybooks`. See the host's documentation for where server
commands are configured. Pass the keystore passphrase via
`TERAKOTA_KEYSTORE_PASSPHRASE_FD` (an open file descriptor — never a plain
environment value).
