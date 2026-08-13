# FAQ

**What is terakota?** A free, local-first command-line tool and MCP server. It
reads the AppFolio, QuickBooks Online, and Dialpad accounts you already run — with
your own credentials, on your own machine — and records every read as a verifiable
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

**Does terakota phone home?** No telemetry, no analytics, no crash reporting —
none, not even optional ones, in any version. The only calls it makes to a service
of ours belong to a production QuickBooks connection made through our connect
service (from v1.4.0): the authorization and the revocation you ask for, plus the
token renewal, which runs on its own whenever the access token nears expiry
(roughly hourly in active use). With no production QuickBooks connection it
contacts no host of ours at all, and its network connections are to the systems
you point it at (AppFolio, Intuit, Dialpad), using your credentials.

**What data can you (the makers) see?** None of your business data, in any mode.
Your AppFolio and Dialpad credentials, your queries, your results, and your receipt
chains stay on your machine — we hold no copy and no interface of ours can reach
them. Reads run from your machine to the vendor directly, always.

If you connect a **production** QuickBooks company through our connect service
(from v1.4.0), we hold two things: your free terakota account, and an eleven-field
connection record — ids, a keyed hash of the realm, status, scope, timestamps, and
a revocation reason; metadata only, never your QuickBooks data, and no raw realm id
or company name. QuickBooks token material transits the broker during authorization
and renewal and comes back sealed to your machine; nothing we could read is at rest
on our side. AppFolio, Dialpad, local reconciliation, `verify-receipts`, and
sandbox QuickBooks under your own registered Intuit application need no account and
put no service of ours in the path. The full field list is in
[PRIVACY.md](legal/PRIVACY.md) §3a.

**Can it change my books?** No. The binaries contain no code paths that write to
the connected systems — read-only is a structural property of the shipped client,
asserted by automated checks at build time. One caution: the QuickBooks OAuth
token itself grants read **and** write (Intuit offers no read-only scope), so the
token stored on your machine is more capable than terakota is. Protect it, and
revoke the connection if you suspect compromise: run
`terakota qbo disconnect --company <id>`, which removes the local material — and for
a production connection made through our connect service, the broker performs the
vendor-side revocation (it holds the client secret). Removing the app inside your
Intuit account always works too. Whichever path you use, an access token already
issued keeps working until it expires. The same caution applies to a Dialpad API
key: it carries whatever scope Dialpad granted it — typically more than read — so
it too is more capable than the tools that use it. Revoke it in your Dialpad
account.

**Why is the account-free QuickBooks path sandbox-only?** Intuit accepts loopback
redirect URIs for sandbox apps only. That path returns the authorization to a
listener on your own machine, so it can reach Intuit **sandbox** companies and
nothing else. A **production** company therefore connects through our hosted
connect service (from v1.4.0), where the exchange and every renewal run server-side
under our registered Intuit application, gated by a free terakota account. AppFolio
and Dialpad reads are unaffected.

**What does "snippet-tier" mean for the Dialpad tools?** That the three Dialpad
read tools — `dialpad_users_list`, `dialpad_offices_list`, and
`dialpad_departments_list` (from v1.5.0) — are verified against a maintainer-held
Dialpad tenant only. Dialpad's developer terms for a distributed third-party client
holding customer-issued keys are recorded and permit it, and issuing a read key is
self-serve on the Pro and Enterprise plans — but that was recorded on one tenant,
where key provisioning did not answer reliably, so treat the plan tier as necessary
and not sufficient. Nothing here claims these tools have run on your account. The
tier is marked in the tool manifest, so a tool cannot quietly graduate itself out
of it.

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
