# terakota Security Advisory & Support Policy

Version 1.2 — Effective 2026-08-13 — applies to the `terakota` and
`verify-receipts` binaries and to the hosted connect service (the broker at
`oauth.terakota.io` and the portal at `app.terakota.io`).

## 1. How we tell you about security problems

Security advisories are published two ways, together: as **GitHub Security
Advisories on the release repository** (which generate notifications for
watchers and feed GitHub's alerting) and as **signed advisory files** in the
repository's `advisories/` directory, verifiable exactly like a download. To be
notified, watch the repository with security alerts enabled. We publish
advisories for: vulnerabilities in our binaries, vulnerabilities inherited from
embedded dependencies that are reachable in our usage, integrity incidents
affecting the release pipeline itself, and security incidents affecting the
connect service.

For the connect service there is a second path, because there is someone to
reach: **if an incident affects your terakota account or a connection you made
through the service, we notify you through the account** (the email on it), in
addition to publishing. That notification path is the reason a production
QuickBooks connection requires an account at all — see the Privacy Notice §3a.

Each advisory states: affected versions, severity (CVSS v4.0 score — with a
v3.1 base score included for tooling compatibility — plus plain-language
impact), whether the flaw is reachable in default configuration, the fixed
version **or "no fix yet" with mitigations** (active exploitation is disclosed
before a fix exists, not after), and workarounds if any. For connect-service
incidents, "affected versions" is replaced by the affected time window and the
affected connections. CVE IDs are requested for High/Critical issues where
CVE-eligible (pipeline-integrity and hosted-service incidents may not be).

**If the release-signing key itself is compromised**, advisory signatures from
it prove nothing — so key rotation and emergencies use an out-of-band path: a
rotation notice signed by the new key (and the old, when it remains safe to
use), pinned on the repository and mirrored at terakota.io/security, which is
the authoritative location in a signing-compromise scenario.

**Withdrawn artifacts:** release artifacts are immutable — never silently
replaced — but an artifact found to be dangerous (compromised, malware-laden,
legally barred) can be withdrawn: it stops being served, its checksum moves to
a signed revocation list, and an advisory explains why. Evidence is preserved;
distribution stops.

## 2. Reporting a vulnerability to us

Email security@terakota.io. Redact credentials, tokens, and production
financial data from reports — a proof of concept against your own
synthetic/sandbox data is ideal. (An encrypted reporting channel will be
published at terakota.io/security; until then, redaction is the control.) We
acknowledge within **3 business days**, aim to triage within **7**, and follow
**coordinated disclosure with a 90-day default window** (negotiable for complex
fixes; shorter if actively exploited). We credit reporters who want credit. No
bug bounty is offered at this time.

**Scope & safe harbor:** in scope — the released binaries, the release
repository, terakota.io, **`oauth.terakota.io` (the connect broker), and
`app.terakota.io` (the portal)**. Out of scope — AppFolio's, Intuit's, and
Dialpad's systems (never test against accounts or systems you don't own; they
have their own programs), social engineering, and physical attacks. Test the
connect service only against your own account and your own QuickBooks company;
do not attempt to reach another user's connection, and do not run volumetric or
denial-of-service tests against it. We will not pursue legal action for
good-faith research within this scope that respects privacy, avoids service
disruption, and gives us the disclosure window; we treat reports as
confidential and use them only to fix the issue and credit you.

## 3. Support windows ("support-until")

Definitions: a **release** is any versioned artifact set we publish from the
release repository (no separate LTS/prerelease channels exist); a release is
**supported** while its support-until date is in the future — meaning it
receives advisories naming it when affected, and security fixes ship as a new
release (we never guarantee a fix for the old artifact itself, only an upgrade
path and honest advisories).

- At publication, each release receives a support-until date of **12 months
  from its release date**, recorded on its release page.
- When the NEXT release ships, the prior release's date is **extended if
  needed** so it ends no sooner than **6 months** after that next release —
  dates only ever move later, never earlier. If no next release ever ships,
  the recorded date stands; product discontinuation, should it ever happen,
  gets its own signed, dated announcement at least 6 months ahead.
- A machine-readable, signed `SUPPORT.json` and an in-binary, user-invoked
  version/advisory check are planned for a future release; until they ship,
  the release pages are the record.
- Release artifacts are immutable once published (withdrawal per Section 1
  aside); fixes ship as new releases only.
- When a release ages out, it keeps working as far as we are concerned —
  nothing is disabled by us, and no build reports anything to us on its own,
  with one stated exception: from terakota v1.4.0, a production QuickBooks
  connection made through our connect service renews its token against our
  broker automatically (EULA §4). Whether an old build still functions against
  vendor APIs is outside our control.
- Support-until dates are a property of **binaries**. The connect service is
  operated, not versioned: it is covered by this policy while it runs, and
  Section 5 states what we commit to for it.

## 4. Scope honesty

This policy covers the binaries we sign and ship and the connect service we
operate. It does not cover: forks or rebuilt binaries; the conduct or
availability of AppFolio, Intuit, or Dialpad APIs; credentials, tokens,
keystores, device keys, or receipt chains on your machines (yours to protect —
see the EULA §3); or AI agents that drive terakota. Receipt chains' evidence
class and its limits are stated in the EULA §2 and the FAQ in the release
repository — advisories cover software defects, not misuse of a correctly
functioning tool.

Two boundaries worth stating plainly, because the connect service changes them.
The token capsule the broker returns is sealed against anyone who can read the
URLs and browser history involved in an authorization; it is not a defense
against an attacker who already controls your machine's processes or keychain.
And a revoked or suspended connection stops new authorizations and stops token
renewal immediately — but an access token already issued keeps working until it
expires, so revocation is fast, not instantaneous.

## 5. The connect service: what we commit to

The connect service is the tier that handles credential material, so it carries
commitments the binaries do not:

- **Incident plan.** We maintain a written plan for the connect service
  covering rotation of the Intuit client secret, an emergency service-wide
  disable, notification of affected users through their accounts, and a
  documented path to reconnect afterwards. Section 1 is how you hear about it.
- **Independent review before we serve anyone else.** An independent
  adversarial security review of the connect service happens before the service
  serves its first real customer. Until then the connect route is not open to
  third parties.
- **No dumps, no live poking.** Production is not instrumented dynamically and
  is not dumped. The client secret lives in the host's secret store and never
  appears in a command line, a repository, or a chat.
- **What the broker is not.** It never terminates business-data traffic, never
  proxies a vendor data API, never accepts customer-authored logic, and is
  never an agent or MCP transport. Its egress is allow-listed to Intuit's
  token/revocation host and our own control store — nothing else.

## 6. Related documents

The EULA §2 and §4 describe what the Software does and every connection it makes
to us. The Privacy Notice §3a describes what the connect service stores, for how
long, and how to delete it. The Portal Account Terms at
`https://app.terakota.io/terms` govern the account and the connect service.

[Change log:
v1.2 — Dialpad is named in the two scope statements, so its absence cannot be
read as an invitation: Dialpad's systems join AppFolio's and Intuit's as out of
scope for testing (§2), and Dialpad's API conduct and availability join what
this policy does not cover (§4). No commitment changes — the Dialpad read path
adds no host of ours, so the advisory classes (§1), the support windows (§3),
and the connect-service commitments (§5) are untouched.
v1.1 — the hosted connect service is brought in scope: safe harbor now covers
`oauth.terakota.io` and `app.terakota.io` with testing rules for a credential
surface (§2); incidents on that service are notified through accounts as well as
published (§1); a new §5 records the incident plan, the independent review before
the first real customer, and the operational fences; the "nothing phones home"
line in §3 gains its one stated exception (automatic token refresh from terakota
v1.4.0); §4 adds the capsule threat boundary and the revocation-versus-expiry
boundary.
v1.0 — first published version.]
