# terakota End User License Agreement and Terms of Use

Version 1.1 — Effective 2026-08-13

This agreement is between you (the individual or entity using the Software) and
Bilans Solutions LLC, a Wyoming limited liability company ("we", "us"). It
governs the `terakota` and `verify-receipts` binaries and accompanying
documentation (the "Software"). If you connect production QuickBooks Online
through our hosted connect service, the Portal Account Terms at
`https://app.terakota.io/terms` also govern your terakota account and that
service — Section 9 explains which document controls what.

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
accounts **you are authorized to access** (AppFolio and QuickBooks Online).

**Version scope.** Production QuickBooks connections are supported **from
terakota v1.4.0 onward**. Releases before v1.4.0 connect to Intuit sandbox
companies only, and nothing of ours is in any of their flows. This agreement is
published before v1.4.0 ships, so read every statement about production
QuickBooks as describing v1.4.0 and later. Version 1.0 of this agreement
promised that if a release added a service of ours to the connection flow, this
agreement and the Privacy Notice would be updated first — this version is that
update.

How much of us is in the path depends on what you connect:

- **AppFolio, local reconciliation, receipts, and `verify-receipts`.** No
  account with us, no service of ours in the path, nothing transmitted to us.
  Reads run from your machine to the vendor directly, with your credentials.
- **QuickBooks Online against an Intuit sandbox company, under your own
  registered Intuit application.** The same: no account, no service of ours in
  the path, nothing transmitted to us. Intuit returns the authorization to a
  loopback listener on your own machine, and the exchange and every renewal run
  there with your own client secret. Intuit accepts loopback redirect URIs for
  sandbox only, so this is the path for a sandbox company — a production
  QuickBooks company connects through our connect service, below.
- **Production QuickBooks Online through our connect service (from terakota
  v1.4.0).** A free terakota account is required, and the initial authorization
  plus every token renewal run through our hosted connect broker at
  `oauth.terakota.io`, under **our** registered Intuit application. The
  authorization code and the token material **transit** that broker. They are
  never stored there.

What is true in every one of those modes: no business data, no query, no query
result, and no AppFolio credential ever reaches us. Your QuickBooks reads run
from your machine to Intuit directly — the connect broker never carries them,
and it never proxies a vendor data API. We are not affiliated with, endorsed by,
or sponsored by AppFolio, Inc. or Intuit Inc.; their services are governed by
your agreements with them.

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
  You are responsible for protecting stored credentials, tokens, and the
  per-install device key on your machines. **Revoking a production connection
  is not instantaneous.** For connections made through our connect service, the
  broker performs the revocation with Intuit (it holds the client secret);
  `terakota qbo disconnect --company <id>` deletes the material on your machine
  and tells you where the vendor-side revocation happens; disconnecting the app
  in your Intuit account remains available to you at any time. Whichever path
  you use, an access token already issued keeps working until it expires.
- **Two ways to connect QuickBooks, and only one needs an account.** The
  default path runs under our registered Intuit application and requires a free
  terakota account. The advanced path uses **your own** registered Intuit
  application against an Intuit sandbox company: it stays account-free, and the
  exchange and every renewal run locally with your own client secret. The
  account requirement exists only for connections that traverse our registered
  vendor application. AppFolio, local reconciliation, receipts,
  `verify-receipts`, and the advanced path stay account-free; extending an
  account requirement to any of them would be a change to this agreement,
  published first.
- **Production QuickBooks depends on a service we run.** Data already cached on
  your machine stays usable offline, and an unexpired access token keeps working
  without us. But the initial authorization, every token renewal, and revocation
  all require the broker. Renewal runs automatically during normal use — the
  access token and its expiry are stored sealed in your local keystore beside
  the refresh token, and the Software renews when the token nears expiry.
  (Intuit's access tokens last about an hour, so in active use renewal is
  roughly hourly.) If the broker is unavailable, fresh production QuickBooks reads stall
  once the access token expires; if it is unavailable for longer than the
  refresh token's lifetime, you have to re-consent. AppFolio and all local
  operation are unaffected. We state this plainly because "AS AVAILABLE" in
  Section 5 now covers a hosted dependency, not just your network.
- **Tokens arrive sealed to your machine.** For connect-service connections,
  the broker performs the code-for-token exchange and returns the tokens as a
  capsule encrypted to a key only your install holds and signed by the broker,
  bound to that one authorization attempt. Your install accepts nothing else.
  The tokens are then stored only on your machine; we keep no plaintext token
  and no token we could decrypt, anywhere, at any time. The sealing protects
  the token against anyone who can read the URLs and browser history involved
  in the authorization; it does not protect against someone who has already
  compromised your machine's processes or keychain.
- **Receipts are integrity records of the Software's executions — with stated
  limits.** For each read, and for each connect, token renewal, and revocation
  made through our connect service, the Software records a declared-intent
  entry before acting and an executed-result entry after, on a hash chain
  stored on your machine; interrupted operations leave a visibly incomplete
  pair. Connections to an Intuit sandbox company under your own registered
  Intuit application record no such pair — no exchange of ours happens, so
  there is nothing of ours to evidence. A connect result records the broker's
  URL and build version, the timestamp, the realm, the granted scope, the fact
  that the exchange executed server-side, and the broker's signed connect
  statement, recorded verbatim; that statement carries no authorization code
  and no token material. The `verify-receipts` tool re-checks a chain's internal
  integrity offline. Receipts are not a record of any AI agent's overall
  activity or reasoning, do not establish completeness of anything beyond the
  Software's own recorded executions, and — because the chain is stored under
  your control — are not tamper-evident against whoever controls the machine and
  files. Treat them as integrity evidence for a chain you have custodied and
  produced, not as third-party attestation.

## 3. Your responsibilities

You will: (a) use the Software only with credentials and accounts you are
authorized to use, and in compliance with your agreements with AppFolio, Intuit,
and any other vendor; (b) comply with applicable law, including privacy and
financial-records law applicable to the data you access; (c) safeguard
credentials, tokens, keystore passphrases, per-install device keys, and receipt
chains stored on your machines — including backing up receipt chains if you rely
on them; (d) validate outputs before relying on them for accounting, legal, or
compliance purposes. The Software retrieves and records data; it does not
provide accounting, legal, audit, or professional advice.

## 4. Updates, advisories, and every connection the Software makes to us

The Software has no automatic updates, no telemetry, no analytics, and no crash
reporting. It contacts exactly two classes of host we operate, and nothing else:

1. **A user-invoked version/advisory check.** It does not exist yet. When it
   ships it will be described in the Privacy Notice before it exists, and it
   will never run automatically.
2. **The connect broker's refresh and revoke calls**, plus the sealed token
   capsule the broker returns to your machine at the end of an authorization
   you started in your browser — for production QuickBooks connections made
   through our connect service (from terakota v1.4.0). The authorization and
   the revocation happen when you ask for them. **The refresh does not — it
   runs automatically during normal use**, whenever the sealed access token
   nears expiry (roughly hourly in active use; both tokens live sealed in your
   local keystore). That is the one place the Software talks to us without you
   asking, and it is why the older "contacts no service of ours on its own"
   wording is gone.

If you have no production connection through our connect service, the Software
makes no call to any host of ours at all. This list is closed: adding another
connection to a service of ours means changing this agreement and the Privacy
Notice first, published and announced on the release repository before it takes
effect.

Security advisories are published on the release repository per the Security
Advisory & Support Policy; watch the repository to be notified. Support expiry
never disables the Software: we build nothing that turns it off. Whether an old
build keeps functioning otherwise depends on factors outside our control (your
systems, vendor APIs, and — for production QuickBooks — the availability of the
connect service described in Section 2), which Section 5 covers.

## 5. No warranty

THE SOFTWARE IS PROVIDED "AS IS" AND "AS AVAILABLE", WITH ALL FAULTS AND WITHOUT
WARRANTY OF ANY KIND. TO THE MAXIMUM EXTENT PERMITTED BY LAW, WE DISCLAIM ALL
WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, INCLUDING MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE, TITLE, NON-INFRINGEMENT, ACCURACY, AND UNINTERRUPTED OR
ERROR-FREE OPERATION. WE DO NOT WARRANT THAT DATA RETRIEVED, RECORDED, OR VERIFIED
BY THE SOFTWARE IS ACCURATE, COMPLETE, OR CURRENT — SOURCE SYSTEMS, NETWORKS, AND
YOUR CONFIGURATION ARE OUTSIDE OUR CONTROL. WE DO NOT WARRANT THAT THE CONNECT
SERVICE DESCRIBED IN SECTION 2 WILL BE AVAILABLE OR UNINTERRUPTED. SOME
JURISDICTIONS DO NOT ALLOW CERTAIN DISCLAIMERS, SO PARTS OF THIS SECTION MAY NOT
APPLY TO YOU.

"AS AVAILABLE" is not decorative here. Section 2 states exactly what stops
working when the connect service is down, and for how long you can keep working
without it.

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

Claims about your terakota account or the connect service are addressed by the
Portal Account Terms, which carry their own limitation of liability; for those
claims, those terms control (Section 9).

## 7. Export compliance

The Software uses standard cryptography (TLS, SHA-256, AES, and — for the
connect-service token capsule — X25519 key agreement, HKDF-SHA256, AES-256-GCM,
with Ed25519 signatures). You may not download, use, or re-export the Software
in or to any jurisdiction or party prohibited under U.S. export laws, including
embargoed destinations and denied-party lists, and you represent you are not on
any such list.

## 8. Termination

This agreement ends automatically if you breach it; you may end it any time by
deleting the Software. Sections 2, 5, 6, 7, and 9 survive, along with Section
3's obligations respecting data already accessed. On termination you must stop
using the Software. Data the Software stored on your machines (keystores,
receipt chains) stays where it is: we claim no rights in it and hold no copy of
it; rights of third parties (your clients, employers, or data sources) in its
contents are unaffected by this agreement.

Ending this agreement does not by itself close your terakota account or delete
its connection records. If you have connected production QuickBooks through our
connect service, write to contact@bilans.io to close the account and an operator
runs the offboarding sequence in the Portal Account Terms. There is no
self-serve close button. Privacy Notice §3a states what is held, what the
closure removes, and what we keep and for how long after a connection is
revoked. Closing the account blocks new connection attempts and
stops token renewals; an access token already issued keeps working until it
expires.

## 9. General

This agreement is the entire agreement about the Software and supersedes prior
discussions. If you use our connect service, the Portal Account Terms at
`https://app.terakota.io/terms` govern your terakota account and that service,
and this agreement governs the Software; where a claim concerns the account or
the connect service, the Portal Account Terms control, and where it concerns the
Software, this agreement controls. The Privacy Notice and the
Security Advisory & Support Policy are referenced disclosures describing our
practices, not contractual obligations, except where this agreement expressly
incorporates a described practice; if they conflict with this agreement, this
agreement governs. It is governed by the laws of Wyoming, excluding
conflict-of-law rules; exclusive venue is Sheridan County, Wyoming. If a
provision is unenforceable, the rest stands. You may not assign this agreement
without our consent; we may assign it to a successor. No waiver is implied. U.S.
Government users: the Software is commercial computer software under FAR 12.212
/ DFARS 227.7202.

Contact: contact@bilans.io

[Change log:
v1.1 — production QuickBooks connections through our hosted connect service
(from terakota v1.4.0): a free terakota account is required for connections
under our registered Intuit application; the broker is named as a permanent
runtime dependency with its degradation stated (§2); the connections the
Software makes to us are now an enumerated closed set, including the automatic
token refresh (§4); token delivery is described as a sealed capsule with its
threat boundary (§2); revocation names the broker as the actor for production
connections and states that an issued access token survives until expiry (§2);
receipts cover connect, token renewal, and revocation on the connect-service
path (§2); termination separates deleting the Software from closing the account
(§8); the Portal Account Terms are named with
explicit precedence (§9). The v1.0 sandbox-only sentence is retired — this
version is the update it promised.
v1.0 — first published version.]
