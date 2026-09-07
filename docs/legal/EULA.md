# terakota End User License Agreement and Terms of Use

Version 1.3 — Effective 2026-09-07

This agreement is between you (the individual or entity using the Software) and
Bilans Solutions LLC, a Wyoming limited liability company ("we", "us"). It
governs the `terakota` and `verify-receipts` binaries and accompanying
documentation (the "Software"). If you hold a terakota account — because you
connected production QuickBooks Online through our hosted connect service,
or because you signed in with `terakota login` to use our control panel
(from terakota `v1.8.0`) — the Portal Account Terms at
`https://app.terakota.io/terms` also govern that account, the connect
service, and the control panel; Section 9 explains which document controls
what.

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
accounts **you are authorized to access** (AppFolio, QuickBooks Online, and
Dialpad).

**Version scope.** Production QuickBooks connections are supported **from
terakota v1.4.0 onward**. Releases before v1.4.0 connect to Intuit sandbox
companies only, and nothing of ours is in any of their flows. Version 1.0 of
this agreement promised that if a release added a service of ours to the
connection flow, this agreement and the Privacy Notice would be updated
first — version 1.1, published before v1.4.0 shipped, was that update. Read
every statement about production QuickBooks as describing v1.4.0 and later.
Signing in (`terakota login`), linking a company to a hosted tenant
(`terakota link`), and the control-plane commands in Section 4 item 3 exist
**from terakota `v1.8.0` onward**; earlier releases have none of them and
make none of those calls. A release may carry fewer of those commands than
Section 4 lists; it never carries a call Section 4 does not.

How much of us is in the path depends on what you connect:

- **AppFolio, local reconciliation, receipts, and `verify-receipts`.** No
  account with us, no service of ours in the path, nothing transmitted to
  us. Reads run from your machine to the vendor directly, with your
  credentials. That stays true if you later sign in and link a company to a
  hosted tenant: the link adds the control-plane commands below, and changes
  nothing about how an AppFolio read runs or what it sends.
- **Dialpad, with an API key you supply (BYO).** No account with us, no
  service of ours in the path, nothing transmitted to us. Reads run from your
  machine to Dialpad directly with your own key, and each one is receipted
  locally like every other read. This surface ships **snippet-tier**: it has
  been verified against a maintainer-held Dialpad tenant only, never on a
  customer account, so treat it as unproven on yours until you have run it.
  Your Dialpad key carries whatever scope Dialpad granted it — typically more
  than read.
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
- **Our control panel, if you sign in and link (from terakota `v1.8.0`) —
  optional, and separate from everything above.** Terakota also runs a
  hosted delivery spine for tenants we onboard. If you are a member of such
  a tenant, `terakota login` signs this binary in to your terakota account
  (through our sign-in provider, in your browser — the Software never
  creates an account on its own), and `terakota link` binds one local
  company to one hosted tenant after a live check that you are a member.
  From then on the `platform-*` commands and `events-tail` read — and, from
  the release that ships them, the four `platform-` change commands alter —
  **our own delivery metadata for that tenant**: which topics exist, which
  subscriptions route them and to which named destination, queue counts,
  held and dead-lettered deliveries (never their content), and an index of
  delivered events (sequence, topic, entity id, event id, times — the index
  has no payload column). A call sends us the linked tenant, the command,
  and the command's own inputs; it never sends a vendor credential, a query,
  a result, a receipt, or your local company id, and nothing you type as
  `--intent` leaves your machine. `terakota unlink` removes the link and
  contacts nothing; `terakota logout` revokes the sign-in and deletes the
  tokens from your machine. **If you never run `terakota login`, nothing
  changes:** the same commands behave the same way, receipts on them are
  identical except for the version stamp, and the Software opens no
  connection to any host of ours beyond Section 4 item 2. A refusal for
  being signed out or unlinked tells you which command clears it and never
  asks you to create an account.

What is true in every one of those modes: no business data, no query, no
query result, and no AppFolio or Dialpad credential ever reaches us, and no
receipt. Your QuickBooks reads run from your machine to Intuit directly —
the connect broker never carries them, and it never proxies a vendor data
API. We are not affiliated with, endorsed by, or sponsored by AppFolio,
Inc., Intuit Inc., or Dialpad, Inc.; their services are governed by your
agreements with them.

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
authorized to use, and in compliance with your agreements with AppFolio,
Intuit, Dialpad, and any other vendor; (b) comply with applicable law,
including privacy and financial-records law applicable to the data you
access; (c) safeguard credentials, tokens (including the sign-in tokens
`terakota login` stores), keystore passphrases, per-install device keys, and
receipt chains stored on your machines — including backing up receipt chains
if you rely on them; (d) validate outputs before relying on them for
accounting, legal, or compliance purposes. The Software retrieves and
records data; it does not provide accounting, legal, audit, or professional
advice.

## 4. Updates, advisories, and every connection the Software makes to us

The Software has no automatic updates, no telemetry, no analytics, and no crash
reporting. It contacts exactly three classes of host — each one ours, or run
for us by the sign-in provider item 3 names — and nothing else:

1. **A user-invoked version/advisory check.** It does not exist yet. When it
   ships it will be described in the Privacy Notice before it exists, and it
   will never run automatically.
2. **The connect broker's refresh and revoke calls**, plus the sealed token
   capsule the broker returns to your machine at the end of an authorization
   you started in your browser — for production QuickBooks connections made
   through our connect service (from terakota v1.4.0). The authorization and
   the revocation happen when you ask for them. **The refresh does not — it
   runs automatically during normal use**, whenever the sealed access token
   nears expiry (roughly hourly in active use; both tokens live sealed in
   your local keystore). That, and the sign-in renewal in item 3, are the
   only two places the Software talks to a host in this list without you
   asking for that particular call — both only while a command you ran is in
   progress; a running `events-tail` (item 3) polls too, but only because
   you started it and only until you stop it. That is why the older
   "contacts no service of ours on its own" wording is gone.
3. **The control plane, after you sign in and link (from terakota
   `v1.8.0`).** Two hosts, sixteen commands, every one of them started by
   you:
   - **Our sign-in host, `dev-bo1prweh.us.auth0.com`, the host our sign-in
     provider (Auth0) runs for us.** `terakota login` asks it for a device
     code, prints a short code and the page to confirm it on, and opens that
     page in your browser unless you tell it not to, then polls the host
     until you approve or the code expires; it returns an access token that
     expires within an hour and a refresh token that rotates on every use, both
     stored only in your local keystore. `terakota logout` asks the same
     host to revoke the refresh token and deletes both tokens from your
     machine. One more call goes there: when a control-plane command in the
     next bullet finds the access token expired, the Software asks the same
     host to renew it with the refresh token before the read — that renewal
     is what rotates the refresh token, and it happens only inside a command
     you ran (a running `events-tail` included). Nothing else in the
     Software contacts that host.
   - **Our control-plane API at `app.terakota.io/api/v1`,** reached with
     that access token and only by these commands: `link` (a live check that
     you are a member of the tenant you name — one `platform-status` read;
     the link itself is written to a profile file on your machine, and
     `unlink` removes it without contacting anyone); the reads
     `platform-status`, `platform-topics`, `platform-subscriptions`,
     `platform-queue-health`, `platform-quarantine` and `platform-dlq`; the
     changes `platform-subscribe`, `platform-unsubscribe`, `platform-replay`
     and `platform-catchall-replay`; and `events-tail`, which polls the
     delivery-event index every few seconds for as long as you leave it
     running — **each poll is one call**, the tail never starts on its own,
     and it stops when you stop it. The MCP tools `platform_status_get`,
     `platform_topics_list`, `platform_subscriptions_list`,
     `platform_queue_health_get` and `platform_events_list` make the same
     reads, one call each; no MCP tool signs in, links, changes routing, or
     runs a tail. `account` (who you are signed in as) reads the entry on
     your machine and calls neither host. A call sends the linked tenant,
     the command, and that command's own inputs (a page position; the
     routing change you asked for; the id of a held delivery to replay). It
     receives our delivery metadata for that tenant — never an event's
     content, an ingest token, a destination's address, or a secret. We
     record these calls on our side (Privacy Notice §3a): a read or a tail
     poll leaves one line — your account, the tenant, which command, the
     time — in a log we delete after 90 days; a change leaves one permanent
     line in the routing audit; `account` leaves nothing. The reads and the
     tail record nothing on your receipt chain, and the four change commands
     are receipted on the linked company's chain from the release that ships
     them.

If you have no production connection through our connect service and have
not signed in with `terakota login`, the Software makes no call to any host
of ours at all. Signing in and linking are optional, and a release may carry
fewer of the item-3 commands than listed — it never carries a call this
Section does not. This list is closed: adding another connection to a
service of ours means changing this agreement and the Privacy Notice first,
published and announced on the release repository before it takes effect.

Security advisories are published on the release repository per the Security
Advisory & Support Policy; watch the repository to be notified. Support expiry
never disables the Software: we build nothing that turns it off. Whether an old
build keeps functioning otherwise depends on factors outside our control
(your systems, vendor APIs, and — for production QuickBooks and for the
control-plane commands — the availability of the connect service and the
control plane described in Section 2), which Section 5 covers.

## 5. No warranty

THE SOFTWARE IS PROVIDED "AS IS" AND "AS AVAILABLE", WITH ALL FAULTS AND WITHOUT
WARRANTY OF ANY KIND. TO THE MAXIMUM EXTENT PERMITTED BY LAW, WE DISCLAIM ALL
WARRANTIES, EXPRESS, IMPLIED, OR STATUTORY, INCLUDING MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE, TITLE, NON-INFRINGEMENT, ACCURACY, AND UNINTERRUPTED OR
ERROR-FREE OPERATION. WE DO NOT WARRANT THAT DATA RETRIEVED, RECORDED, OR VERIFIED
BY THE SOFTWARE IS ACCURATE, COMPLETE, OR CURRENT — SOURCE SYSTEMS, NETWORKS, AND
YOUR CONFIGURATION ARE OUTSIDE OUR CONTROL. WE DO NOT WARRANT THAT THE CONNECT SERVICE
OR THE CONTROL PLANE DESCRIBED IN SECTION 2 WILL BE AVAILABLE OR UNINTERRUPTED. SOME
JURISDICTIONS DO NOT ALLOW CERTAIN DISCLAIMERS, SO PARTS OF THIS SECTION MAY NOT APPLY
TO YOU.

"AS AVAILABLE" is not decorative here. Section 2 states exactly what stops
working when the connect service is down, and for how long you can keep
working without it. When the control plane is unreachable, the control-plane
commands refuse with a typed error and nothing local is affected — every
other command keeps working.

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

Claims about your terakota account, the connect service, or the control
panel are addressed by the Portal Account Terms, which carry their own
limitation of liability; for those claims, those terms control (Section 9).

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

Ending this agreement does not by itself close your terakota account or
delete its connection records, and neither does `terakota logout` — that
revokes the sign-in and deletes the tokens from your machine, nothing more.
If you hold an account, write to contact@bilans.io to close it and an
operator runs the offboarding sequence in the Portal Account Terms. There is
no self-serve close button. Privacy Notice §3a states what is held, what the
closure removes, and what we keep and for how long after a connection is
revoked. Closing the account blocks new connection attempts, stops token
renewals, and closes the control panel to you; an access token already
issued keeps working until it expires, and routing changes you already made
on a tenant stay as they are until an operator or another member changes
them.

## 9. General

This agreement is the entire agreement about the Software and supersedes prior
discussions. If you hold a terakota account, the Portal Account Terms at
`https://app.terakota.io/terms` govern that account, the connect service, and
the control panel, and this agreement governs the Software; where a claim
concerns the account, the connect service, or the control panel, the Portal
Account Terms control, and where it concerns the Software, this agreement
controls. The Privacy Notice and the Security Advisory & Support Policy are
referenced disclosures describing our practices, not contractual obligations,
except where this agreement expressly incorporates a described practice; if
they conflict with this agreement, this agreement governs. It is governed by
the laws of Wyoming, excluding conflict-of-law rules; exclusive venue is
Sheridan County, Wyoming. If a provision is unenforceable, the rest stands.
You may not assign this agreement without our consent; we may assign it to a
successor. No waiver is implied. U.S. Government users: the Software is
commercial computer software under FAR 12.212 / DFARS 227.7202.

Contact: contact@bilans.io

[Change log:
v1.3 — the optional terakota account gains a second purpose, our control
panel, and the Software gains an optional bridge to it (from terakota
`v1.8.0`): `terakota login` (sign-in through our sign-in host),
`terakota link` (one local company ↔ one hosted tenant), the `platform-*`
reads and change commands, and `events-tail` (§2, §4). Section 4's
enumerated closed set grows from two classes to three and names our sign-in
host, `app.terakota.io/api/v1`, and every command that may call them, each
poll of a running tail included; the "makes no call to any host of ours at
all" sentence is now conditioned on not having signed in. What a
control-plane call sends and receives is stated — our delivery metadata,
never an event's content, never anything of yours (§2). Nothing about
AppFolio, Dialpad, local operation, receipts, `verify-receipts`, or the
connect service changes; unlinked use is unchanged and said so (§2).
Termination separates `logout` from closure and states that closure leaves
routing in place (§8); the Portal Account Terms now govern the control panel
with the same precedence (§1, §6, §9).
v1.2 — Dialpad added to the read surface (§2): reads run from your machine to
Dialpad directly with a Dialpad API key you supply, receipted locally, with no
account with us and no service of ours in the path; the surface is disclosed as
snippet-tier, verified against a maintainer-held tenant only and never on a
customer account. Dialpad, Inc. is named in the non-affiliation sentence (§2)
and in your vendor-agreement responsibility (§3(a)). **Section 4's enumerated
closed set of connections the Software makes to us is UNCHANGED by this
version: Dialpad adds no host of ours, and nothing in the Dialpad path
contacts us.**
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
