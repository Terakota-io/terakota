# FAQ

**What is terakota?** A free, local-first command-line tool and MCP server. It
reads the AppFolio and QuickBooks Online accounts you already run — with your own
credentials, on your own machine — and records every read as a verifiable
integrity receipt. It comes with `verify-receipts`, a standalone tool that checks
those receipts offline.

**What are receipts — and what are they not?** For each read, terakota records a
declared-intent entry before it acts and an executed-result entry after, linked on
a hash chain stored on your machine. They are **integrity records of what terakota
executed**, and you can verify a chain offline with `verify-receipts`. Their
stated limits:

- They are **not a log of an AI agent's full activity** or reasoning — only of the
  reads terakota itself performed.
- They are **not tamper-evident against whoever controls the machine.** The chain
  is stored under your control, so treat receipts as integrity evidence for a
  chain you custodied and produced — not as third-party attestation.

**Does terakota phone home?** No. There is no telemetry, no analytics, no crash
reporting, and no network call to any service we operate — none, not even optional
ones, in this version. terakota's only network connections are to the systems you
point it at (AppFolio, Intuit), using your credentials.

**What data can you (the makers) see?** Nothing. terakota needs no account with
us and contacts no service of ours. Your credentials, tokens, queries, results,
and receipt chains stay on your machine; we have no way to access them and hold no
copy. See [PRIVACY.md](legal/PRIVACY.md).

**Can it change my books?** No. The binaries contain no code paths that write to
the connected systems — read-only is a structural property of the shipped client,
asserted by automated checks at build time. One caution: the QuickBooks OAuth
token itself grants read **and** write (Intuit offers no read-only scope), so the
token stored on your machine is more capable than terakota is. Protect it, and
revoke it through Intuit if you suspect compromise.

**Why does QuickBooks only work against a sandbox?** In v1, QuickBooks Online
connections run against Intuit **sandbox** companies only. Production QuickBooks
arrives in a later release. AppFolio reads are unaffected.

**Why isn't the binary signed by my OS?** At v1 the binaries are cosign-signed
(see [verify.md](verify.md)) but not yet OS code-signed, so macOS Gatekeeper or
Windows SmartScreen may warn that the publisher is unverified. That is expected;
OS code-signing arrives in a later release. [install.md](install.md) shows how to
clear the warnings after you have verified the download.

**How do I report a security problem?** Email security@terakota.io and follow the
[Security Advisory & Support Policy](../SECURITY.md). Redact credentials, tokens,
and any production financial data — a proof of concept against your own
sandbox/synthetic data is ideal.

**What license are the binaries under?** The binaries are covered by the
[EULA](legal/EULA.md) — a free license to use them, unmodified. The open-source
components embedded in them are licensed under their own terms; those notices ship
in the `THIRD_PARTY_NOTICES` file in each archive and are also printed by
`terakota licenses`.

**What does it cost?** The `terakota` binary and the `verify-receipts` verifier
are free.
