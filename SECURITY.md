# terakota Security Advisory & Support Policy

Version 1.0 — Effective 2026-07-24 — applies to the `terakota` and
`verify-receipts` binaries.

## 1. How we tell you about security problems

Security advisories are published two ways, together: as **GitHub Security
Advisories on the release repository** (which generate notifications for
watchers and feed GitHub's alerting) and as **signed advisory files** in the
repository's `advisories/` directory, verifiable exactly like a download. To be
notified, watch the repository with security alerts enabled. We publish
advisories for: vulnerabilities in our binaries, vulnerabilities inherited from
embedded dependencies that are reachable in our usage, and integrity incidents
affecting the release pipeline itself.

Each advisory states: affected versions, severity (CVSS v4.0 score — with a
v3.1 base score included for tooling compatibility — plus plain-language
impact), whether the flaw is reachable in default configuration, the fixed
version **or "no fix yet" with mitigations** (active exploitation is disclosed
before a fix exists, not after), and workarounds if any. CVE IDs are requested
for High/Critical issues where CVE-eligible (pipeline-integrity incidents may
not be).

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
repository, and terakota.io. Out of scope — AppFolio's and Intuit's systems
(never test against accounts or systems you don't own; they have their own
programs), social engineering, and physical attacks. We will not pursue legal
action for good-faith research within this scope that respects privacy, avoids
service disruption, and gives us the disclosure window; we treat reports as
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
  nothing phones home and nothing is disabled by us — but it no longer
  receives advisories tailored to it. Whether an old build still functions
  against vendor APIs is outside our control.

## 4. Scope honesty

This policy covers the binaries we sign and ship. It does not cover: forks or
rebuilt binaries; the conduct or availability of AppFolio or Intuit APIs;
credentials, tokens, keystores, or receipt chains on your machines (yours to
protect — see the EULA §3); or AI agents that drive terakota. Receipt chains'
evidence class and its limits are stated in the EULA §2 and the FAQ in the
release repository — advisories cover software defects, not misuse of a
correctly functioning tool.

[Change log: v1.0 — first published version.]
