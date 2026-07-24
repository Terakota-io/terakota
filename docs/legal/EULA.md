# terakota End User License Agreement and Terms of Use

Version 1.0 — Effective 2026-07-24

This agreement is between you (the individual or entity using the Software) and
Bilans Solutions LLC, a Wyoming limited liability company ("we", "us"). It
governs the `terakota` and `verify-receipts` binaries and accompanying
documentation (the "Software").

**BY DOWNLOADING, INSTALLING, OR USING THE SOFTWARE, YOU AGREE TO THIS AGREEMENT.
IF YOU DO NOT AGREE, DO NOT USE THE SOFTWARE.** If you use the Software on behalf
of an entity, you represent that you have authority to bind that entity, and
"you" includes that entity and its authorized personnel.

## 1. License

We grant you a free, non-exclusive, non-transferable license to install and use
the Software, in unmodified binary form, for your internal business or personal
purposes, including copies for backup and internal deployment. You may
redistribute the Software only as complete, unmodified official release
artifacts (archives or their contained binaries, with all notices intact and
checksums unaltered) — this permits mirrors, package managers, and internal
caches; it does not permit modified builds, or removing this agreement or any
notice. You may not: (a) modify, adapt, or create derivative works of the
Software; (b) reverse engineer, decompile, or disassemble the Software except to
the extent this restriction is prohibited by applicable law or covered by a
license we publish for corresponding source code; (c) sell the Software or host
it as a service for third parties without our written permission; (d) remove or
alter any notices in the Software or its archives. We and our licensors retain
all rights not expressly granted. Open-source components embedded in the
Software are licensed under their own terms (see the THIRD_PARTY_NOTICES file in
each archive); nothing in this agreement limits your rights under those
licenses.

## 2. What the Software does — and does not do (read this)

The Software runs on your machine, with credentials **you** supply, against
accounts **you are authorized to access** (AppFolio and QuickBooks Online). It
requires no account with us, contacts no service we operate, sends us nothing,
and we cannot see your credentials, queries, results, or receipts. Its only
network connections are to the systems you connect it to, using your
credentials. QuickBooks support in this version operates against Intuit sandbox
companies; production QuickBooks connections arrive in a future release, and if
that release adds any service of ours to the connection flow, this agreement
and the Privacy Notice will be updated first. We are not affiliated with,
endorsed by, or sponsored by AppFolio, Inc. or Intuit Inc.; their services are
governed by your agreements with them.

- **Read-only toward your business systems, by construction.** The Software
  contains no code paths that write to the connected business systems; this is
  a property of the shipped client code, asserted by automated checks at build
  time. It is a claim about this Software as built — not about other software
  on your machine, not a guarantee against defects or vulnerabilities, and not
  a property of your credentials.
- **Your credentials can be more powerful than the tool.** Intuit's
  `accounting` OAuth scope grants read AND write — Intuit offers no read-only
  scope — so the QuickBooks token stored on your machine can do more than the
  Software ever will. The same caution applies to any credential you supply.
  You are responsible for protecting stored credentials and tokens and for
  revoking them through the vendor (e.g., disconnecting the app in your Intuit
  account) if you suspect compromise.
- **Receipts are integrity records of the Software's executions — with stated
  limits.** For each read, the Software records a declared-intent entry before
  acting and an executed-result entry after, on a hash chain stored on your
  machine; interrupted operations leave a visibly incomplete pair. The
  `verify-receipts` tool re-checks a chain's internal integrity offline.
  Receipts are not a record of any AI agent's overall activity or reasoning,
  do not establish completeness of anything beyond the Software's own recorded
  executions, and — because the chain is stored under your control — are not
  tamper-evident against whoever controls the machine and files. Treat them as
  integrity evidence for a chain you have custodied and produced, not as
  third-party attestation.

## 3. Your responsibilities

You will: (a) use the Software only with credentials and accounts you are
authorized to use, and in compliance with your agreements with AppFolio, Intuit,
and any other vendor; (b) comply with applicable law, including privacy and
financial-records law applicable to the data you access; (c) safeguard
credentials, tokens, keystore passphrases, and receipt chains stored on your
machines — including backing up receipt chains if you rely on them; (d) validate
outputs before relying on them for accounting, legal, or compliance purposes.
The Software retrieves and records data; it does not provide accounting, legal,
audit, or professional advice.

## 4. Updates and advisories

The Software has no automatic updates and contacts no service of ours on its
own. Security advisories are published on the release repository per the
Security Advisory & Support Policy; watch the repository to be notified. A
user-invoked version/advisory check command may ship in a future release — it
will be described in the Privacy Notice before it exists, and it will never run
automatically. Support expiry never disables the Software: we build nothing
that turns it off. Whether an old build keeps functioning otherwise depends on
factors outside our control (your systems, vendor APIs), which Section 5
covers.

## 5. No warranty

THE SOFTWARE IS PROVIDED "AS IS" AND "AS AVAILABLE", WITH ALL FAULTS AND WITHOUT
WARRANTY OF ANY KIND. TO THE MAXIMUM EXTENT PERMITTED BY LAW, WE DISCLAIM ALL
WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, INCLUDING MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE, TITLE, NON-INFRINGEMENT, ACCURACY, AND UNINTERRUPTED OR
ERROR-FREE OPERATION. WE DO NOT WARRANT THAT DATA RETRIEVED, RECORDED, OR VERIFIED
BY THE SOFTWARE IS ACCURATE, COMPLETE, OR CURRENT — SOURCE SYSTEMS, NETWORKS, AND
YOUR CONFIGURATION ARE OUTSIDE OUR CONTROL. SOME JURISDICTIONS DO NOT ALLOW CERTAIN
DISCLAIMERS, SO PARTS OF THIS SECTION MAY NOT APPLY TO YOU.

## 6. Limitation of liability

TO THE MAXIMUM EXTENT PERMITTED BY LAW: (A) NEITHER WE NOR OUR SUPPLIERS WILL BE
LIABLE FOR ANY INDIRECT, INCIDENTAL, SPECIAL, CONSEQUENTIAL, OR PUNITIVE DAMAGES, OR
FOR LOST PROFITS, LOST DATA, BUSINESS INTERRUPTION, OR THE COST OF SUBSTITUTE
SERVICES, ARISING FROM OR RELATED TO THE SOFTWARE, UNDER ANY THEORY (CONTRACT, TORT,
NEGLIGENCE, OR OTHERWISE), EVEN IF ADVISED OF THE POSSIBILITY; AND (B) OUR TOTAL
AGGREGATE LIABILITY WILL NOT EXCEED ONE HUNDRED U.S. DOLLARS (US $100). THE SOFTWARE
IS FREE; THIS ALLOCATION OF RISK IS A CONDITION OF PROVIDING IT WITHOUT CHARGE.
NOTHING IN THIS AGREEMENT LIMITS LIABILITY THAT CANNOT BE LIMITED UNDER APPLICABLE
LAW.

## 7. Export compliance

The Software uses standard cryptography (TLS, SHA-256, AES). You may not
download, use, or re-export the Software in or to any jurisdiction or party
prohibited under U.S. export laws, including embargoed destinations and
denied-party lists, and you represent you are not on any such list.

## 8. Termination

This agreement ends automatically if you breach it; you may end it any time by
deleting the Software. Sections 2, 5, 6, 7, and 9 survive, along with Section
3's obligations respecting data already accessed. On termination you must stop
using the Software. Data the Software stored on your machines (keystores,
receipt chains) stays where it is: we claim no rights in it and never have a
copy; rights of third parties (your clients, employers, or data sources) in its
contents are unaffected by this agreement.

## 9. General

This agreement is the entire agreement about the Software and supersedes prior
discussions. The Privacy Notice and the Security Advisory & Support Policy are
referenced disclosures describing our practices, not contractual obligations,
except where this agreement expressly incorporates a described practice; if they
conflict with this agreement, this agreement governs. It is governed by the laws
of Wyoming, excluding conflict-of-law rules; exclusive venue is Sheridan County, Wyoming.
If a provision is unenforceable, the rest stands. You may not assign this
agreement without our consent; we may assign it to a successor. No waiver is
implied. U.S. Government users: the Software is commercial computer software
under FAR 12.212 / DFARS 227.7202.

Contact: legal@terakota.io

[Change log: v1.0 — first published version.]
