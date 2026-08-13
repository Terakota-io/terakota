# terakota

**terakota** is a free, local-first command-line tool and MCP server that reads the
AppFolio, QuickBooks Online, and Dialpad accounts you already run — using your own
credentials, on your own machine — and records every read as a verifiable
integrity receipt.

It is read-only by construction. AppFolio reads, Dialpad reads, local work, and
`verify-receipts` send nothing to us and need no account with us; connecting a
**production** QuickBooks company (from v1.4.0) runs through a hosted connect
service we operate and needs a free terakota account. It ships with
`verify-receipts`, a standalone tool that checks receipt chains offline.

## What it does

- **Reads your systems with your credentials.** terakota connects to AppFolio,
  QuickBooks Online, and Dialpad using credentials you supply, against accounts you
  are authorized to access. Reads always run from your machine to the vendor
  directly. The only other host it ever talks to is our connect service, and only
  if you connect a production QuickBooks company (from v1.4.0) — for authorization,
  token renewal, and revocation, never for your data.
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
- **Counts and rollups without the records** *(since v1.3.0)*. Every list tool takes
  `--aggregate count`, `--group-by <fields>`, and `--sum <field>`: terakota walks the
  pages and releases derived rows — counts, cross-tabs, sums — instead of record
  bodies. A portfolio count moves kilobytes, not megabytes, and no personal data
  needs to leave the source system to answer "how many". Results carry their own
  denominators (`matched`, per-field population) so a sparse field can't masquerade
  as complete.
- **One-command GL month-close** *(since v1.3.0)*. `appfolio_gl_details_list
  --aggregate count` with a posting-date window sweeps the source's change-window
  constraint internally and returns per-account debit/credit rollups with an explicit
  list of swept, unswept, and failed windows — a total that is missing data says so,
  every time.
- **Field discovery without pulling records** *(since v1.3.1)*. Every tool's `--help`
  (and MCP description) lists the collection's record fields with types and
  live-verified notes — so learning a field name for `--group-by` or `--sum` costs
  zero vendor calls and zero personal data, instead of a full page of record bodies.
- **Whole-walk totals in one invocation** *(since v1.3.1)*. Aggregate walks fold
  page-by-page (the page cap is real now: full-table counts and histograms are
  typically ONE call), partial slices label themselves as subtotals, and a
  multi-slice walk's final slice reports exact merged `walk_totals`. Every
  `gl_details` aggregate — not just the sweep — carries `total_debits`/`total_credits`,
  scope-labeled whenever coverage is incomplete.
- **Dialpad reads with a key you supply** *(since v1.5.0)*. `terakota dialpad
  connect` seals a Dialpad API key of your own into your OS keychain, and
  `dialpad_users_list`, `dialpad_offices_list`, and `dialpad_departments_list`
  read against it — from your machine to Dialpad directly, no service of ours in
  the path, no account with us, each read receipted on your local chain like every
  other read. This family ships **snippet-tier**: it is verified against a
  maintainer-held Dialpad tenant, not yet validated on customer accounts. Nothing
  here claims these tools have run on yours.
- **Per-tool help.** `terakota <tool> --help` prints that tool's own parameters,
  bounds, and the source-system quirks it absorbs; every error carries a
  `next_action` telling you whether and how to retry.
- **No telemetry, ever.** No telemetry, no analytics, no crash reporting — in any
  version, in any mode. Your credentials, queries, results, and receipts stay on
  your machine: we hold no copy, and no interface of ours can reach them. The only
  calls terakota makes to a service of ours are a production QuickBooks
  connection's authorization, token renewal, and revocation (from v1.4.0) —
  renewal runs on its own, roughly hourly while you work. With no production
  QuickBooks connection it contacts no host of ours at all.

> **Two ways to connect QuickBooks (from v1.4.0).** A **production** company
> connects through our hosted connect service at `oauth.terakota.io` and needs a
> free terakota account — Intuit refuses loopback redirects on production apps, so
> the exchange and every renewal have to run server-side under our registered
> Intuit application. The account-free path is your **own** registered Intuit
> application against an Intuit **sandbox** company; loopback is all it can use, so
> it is sandbox-only. Either way your QuickBooks reads run from your machine to
> Intuit directly, and AppFolio and Dialpad are unaffected.

We are not affiliated with, endorsed by, or sponsored by AppFolio, Inc., Intuit
Inc., or Dialpad, Inc.; their services are governed by your agreements with them.

## Get started

On macOS or Linux:

    brew install terakota-io/tap/terakota
    # or
    curl -fsSL https://terakota.io/install.sh | sh

On Windows, download the signed `.zip` from
[Releases](../../releases). Then:

- **Install — all channels and the manual path:** [docs/install.md](docs/install.md)
- **Use it inside Claude Desktop (`.mcpb` extension), start to finish:**
  [docs/install.md](docs/install.md#use-terakota-as-an-mcp-extension)
- **Verify your download:** [docs/verify.md](docs/verify.md)
- **FAQ — what receipts do and do not prove:** [docs/faq.md](docs/faq.md)

## What ships here

Signed release artifacts only — the `terakota` binary and the `verify-receipts`
verifier for Linux, macOS, and Windows (`amd64` + `arm64`). Each release carries
SHA-256 checksums, cosign signatures, SPDX SBOMs, and SLSA build provenance. The
binaries are cosign-signed but not yet OS code-signed at v1 (see
[docs/install.md](docs/install.md) for the expected Gatekeeper/SmartScreen
notices). Source code is not published here.

## Privacy

There is no telemetry, no analytics, and no crash reporting, in any version. Your
credentials, your queries, your results, and your receipts stay on your machine — we
hold no copy, and no interface of ours can reach them.

For AppFolio, Dialpad, local reconciliation, `verify-receipts`, and QuickBooks against
an Intuit sandbox company under your own registered Intuit application, that is the
whole story: nothing transmitted to us, no service of ours in the path, no account to
create.

Connecting a **production** QuickBooks company (from v1.4.0) is the one exception. It
runs through our hosted connect service and needs a free terakota account, and we then
hold that account plus an eleven-field connection record — metadata only, never your
QuickBooks data, and no token we could read at rest. The reads themselves still run from
your machine to Intuit directly; the connect service carries authorization, renewal, and
revocation, and never a vendor data call.

Your credentials rest in your operating system's keychain (macOS Keychain, Windows
Credential Manager, or a Linux Secret Service) and never travel through command
arguments or environment values. Receipts are written to a hash chain on your own
machine.

Full notice: **[docs/legal/PRIVACY.md](docs/legal/PRIVACY.md)** ·
**https://terakota.io/privacy**

## Legal and support

- **License for the binaries:** [docs/legal/EULA.md](docs/legal/EULA.md) — free to
  use, unmodified. Third-party notices ship in each archive and via
  `terakota licenses`.
- **Privacy:** [docs/legal/PRIVACY.md](docs/legal/PRIVACY.md)
- **Security advisories & support policy:** [SECURITY.md](SECURITY.md) — how we
  disclose problems, support windows, and how to report a vulnerability. Signed
  advisories publish under [advisories/](advisories/); watch this repository to be
  notified.
