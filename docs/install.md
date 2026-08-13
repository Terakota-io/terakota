# Install

Installing needs no account and no sign-up, and the binaries carry no telemetry,
analytics, or crash reporting. (Connecting a **production** QuickBooks company —
from v1.4.0 — needs a free terakota account; see [First run](#first-run).) Pick a
channel:

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
    curl -fsSL https://terakota.io/install.sh | TERAKOTA_VERSION=v1.5.0 sh

    # install system-wide (needs write access to /usr/local)
    curl -fsSL https://terakota.io/install.sh | sudo PREFIX=/usr/local sh

**Linux packages (`.deb` / `.rpm`, since v1.1.0)**

    sudo dpkg -i terakota_v1.5.0_linux_amd64.deb
    sudo rpm -i  terakota_v1.5.0_linux_amd64.rpm

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

## Upgrading

Upgrade through the channel you installed with:

    brew upgrade terakota                           # Homebrew
    curl -fsSL https://terakota.io/install.sh | sh  # install script — re-run it
    sudo dpkg -i terakota_v1.5.0_linux_amd64.deb    # .deb — installs over the old
    sudo rpm -U  terakota_v1.5.0_linux_amd64.rpm    # .rpm — -U, not -i, to upgrade

For the MCP extension, download the new `.mcpb` for your platform and install it
from Claude Desktop → **Settings** → **Extensions** again; it replaces the old
bundle. For a manual install, download the new archive, verify it per
[verify.md](verify.md), and overwrite the binaries where you put them.
`terakota version` tells you what you are running.

Your credentials and your receipt chains carry over untouched. They live in your
OS keychain and in terakota's own home directory — installing a release replaces
binaries and nothing else. Coming from v1.0.0 or v1.1.0 is the one case with a
step attached: see the keystore note under
[Where your credentials live](#where-your-credentials-live).

**Expect the first-run notice one more time after upgrading to v1.5.0.** It
reprints when the terms version it pins changes, and v1.5.0 moves from terms 1.1
to 1.2 — so that single reprint is the notice doing its job. It does not print
again on an upgrade that leaves the terms unchanged. `terakota about` shows it
any time.

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

- Linux / macOS: `.tar.gz` (e.g. `terakota_v1.5.0_linux_amd64.tar.gz`,
  `terakota_v1.5.0_darwin_arm64.tar.gz`)
- Windows: `.zip` (e.g. `terakota_v1.5.0_windows_amd64.zip`)

> **Two ways to connect QuickBooks (from v1.4.0).** A **production** company
> connects through our hosted connect service and needs a free terakota account.
> The account-free path is your **own** registered Intuit application against an
> Intuit **sandbox** company — Intuit accepts the loopback redirect it uses for
> sandbox only. AppFolio and Dialpad reads are unaffected, and everything is
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

    tar -xzf terakota_v1.5.0_darwin_arm64.tar.gz
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
    terakota qbo connect --company mybooks         # optional: connect QuickBooks Online
    terakota dialpad connect --company mybooks     # optional: connect Dialpad (BYO API key)

`<yourdomain>.appfolio.com` is your web portal / Reports API address. The Database API
terakota speaks lives at `api.appfolio.com/api/v0` for every customer.

`qbo connect` has two paths (from v1.4.0). A **production** company connects through our
hosted connect service at `oauth.terakota.io` and needs a free terakota account at
`app.terakota.io` — Intuit refuses loopback redirects on production apps, so the exchange
and every renewal run server-side under our registered Intuit application. An Intuit
**sandbox** company can be connected under your **own** registered Intuit application
instead, with the exchange and every renewal running locally against a loopback listener;
that path needs no account. Reads run from your machine to Intuit directly either way.
Disconnect with `terakota qbo disconnect --company mybooks`.

`dialpad connect` (from v1.5.0) is simpler: no OAuth ceremony, no account with us. It
prompts no-echo for a Dialpad API key you issue in your own Dialpad account and seals it
into the keystore alongside the company's other credentials — that key is the whole
binding, and there is no realm to bind. Reads run from your machine to Dialpad directly.
Two things to know before you rely on it. The key carries whatever scope Dialpad granted
it — typically more than read — so it is more capable than the tools that use it. And the
family ships snippet-tier: verified against a maintainer-held Dialpad tenant, not yet
validated on customer accounts. Issuing a read key is self-serve on Dialpad's Pro and
Enterprise plans, but that was recorded on one tenant, so treat the plan tier as
necessary and not sufficient.

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
