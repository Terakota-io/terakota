# terakota Privacy Notice

Version 1.3 — Effective 2026-09-07 — Bilans Solutions LLC

The short version, split by what you connect:

- **AppFolio, Dialpad, local reconciliation, receipts, and
  `verify-receipts`:** the terakota software sends us nothing about them and
  requires no account for them.
- **Production QuickBooks connections (from terakota v1.4.0):** these run
  through a connect service we operate, and they require a free terakota
  account.
- **Our control panel, if you choose to sign in and link (from terakota
  `v1.8.0`):** `terakota login` and `terakota link` are optional; once
  linked, the `platform-*` commands (in `v1.8.0`, the six reads) and
  `events-tail` call our control plane and we record each call. They carry our own delivery metadata for the
  tenant you linked — never your data.

This notice describes every place our infrastructure can observe anything at
all: downloads, the connect service (the broker at `oauth.terakota.io` and
the portal at `app.terakota.io`), the control plane (the portal's `/api/v1`
and our sign-in host), the website, and correspondence you send us.

## 1. The software: what it sends, and to whom

`terakota` and `verify-receipts` run on your machine. They have no telemetry, no
crash reporting, and no analytics — none, in any version, in any mode. There is
no background reporting of any kind.

The calls the software makes to us are a closed, enumerated set of exactly
three classes:

1. **A user-invoked version/advisory check.** It does not exist yet. When it
   ships it will be described here before it exists, and it will never run
   automatically.
2. **The connect broker's refresh and revoke calls**, plus the sealed token
   capsule the broker returns to your machine at the end of an authorization
   you started in your browser — for production QuickBooks connections made
   through our connect service (from terakota v1.4.0). The authorization and
   the revocation happen when you ask for them. The refresh runs automatically
   during normal use, whenever the sealed access token nears expiry — roughly
   hourly in active use; both tokens are stored sealed in your local keystore.
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
     record these calls on our side (Section 3a): a read or a tail poll
     leaves one line — your account, the tenant, which command, the time —
     in a log we delete after 90 days (if that line cannot be written, the
     read still completes and the failure is noted in our application log —
     the tenant, the command and the error, never your account id); a change
     leaves one permanent line in the routing audit; `account` leaves
     nothing. The reads and the tail record nothing on your receipt chain,
     and the four change commands are receipted on the linked company's
     chain from the release that ships them. What each call carries is
     stated in Section 3a, together with what we record about it and for how
     long.

AppFolio, Dialpad, local reconciliation, receipts, `verify-receipts`, and an
Intuit sandbox company under your own registered Intuit application need no
account. If those are all you use and you have not signed in with
`terakota login`, the software makes no call to any host of ours at all. If you
do sign in and link a company, the only calls added are the control-plane calls
in item 3 — an AppFolio or Dialpad read still runs from your machine to the
vendor and still sends us nothing. Signing in and linking are optional, and a
release may carry fewer of the item-3 commands than listed — it never carries a
call this notice does not.

**What is never transmitted to us, in any mode:** your AppFolio and Dialpad
credentials, your queries, your query results, and your receipt chains. They
are stored and processed only on your machines, we hold no copy of them, and
no interface of ours can reach them.

**What does transit us, and only for production QuickBooks connections made
through our connect service:** the Intuit authorization code and the OAuth token
material. The broker performs the code-for-token exchange and each renewal
because Intuit's token endpoint requires a client secret that a downloadable
binary cannot carry. The tokens come back to your install sealed — encrypted to
a key only your machine holds and signed by the broker — and they are stored
only on your machine. Nothing of that material is written down on our side: no
token, no authorization code, no realm id at rest, and none of them in any log,
trace, or metric. Section 3a states exactly what our connect service does keep.
Your QuickBooks data itself never goes near the broker: reads run from your
machine to Intuit directly.

**What a control-plane call sends, and only after you sign in and link (from
terakota `v1.8.0`):** the tenant you linked, the command, and that command's
own inputs — a page position for `events-tail`, the routing change you asked
for, the id of a held delivery to replay — under a sign-in token that
identifies your account. Not your local company id, not the `--intent` text
you typed, and never a vendor credential, a query, a result, or a receipt.
What comes back is our delivery metadata for that tenant: topic names,
subscription bindings and destination names, counts, and for delivered
events their sequence, topic, entity id, event id, and times. There is no
payload column in the index those events are read from, so no event content
can come back.

Where the software stores local data (keystore, receipt chains, config) and how
to delete it is documented in the release repository — deletion is yours to
perform; we hold no copy.

If a future release adds any feature that contacts us beyond the three classes
of call enumerated above, this notice will be updated first: the effective date
above will move, a change note will be added below, and the change will be
announced on the release repository before it takes effect.

## 2. Downloads

Releases are distributed via GitHub. When you download, GitHub processes your
request under its own privacy policy; we see only the aggregate download counts
GitHub exposes to repository owners — no identities, no IP addresses.

## 3. The website and our hosted surfaces

terakota.io is a static informational site served by Cloudflare. It uses
cookieless, aggregate analytics (Cloudflare Web Analytics) — no cookies, no
cross-site tracking, no user identification. It loads fonts from Google Fonts,
so your IP address transits to Google when a font loads. Hosting-level logs are
handled by Cloudflare under its own policies; we retain no server logs of our
own. No accounts exist on the website and no forms collect personal information.

Two further surfaces are ours, both hosted on Fly.io in one US region (iad,
Ashburn, Virginia):

- **`oauth.terakota.io` — the connect broker.** TLS-only, HSTS. It reads no
  cookies, serves no analytics and no third-party scripts, holds no token,
  authorization code, or realm id at rest, and logs none of them. Its metrics
  are bounded outcome classes and aggregate counts — never a URL, body,
  authorization code, token, state value, capsule, or realm id.
- **`app.terakota.io` — the portal and, for accounts an operator has
  switched on, the control panel.** Sign-in gated (Auth0). This is where a
  terakota account lives, where a connection is started, and — from portal
  revision `2026-09-07` — where a member of a hosted tenant sees and changes
  that tenant's delivery routing. Its `/api/v1` is the same panel for the
  terakota binary (from terakota `v1.8.0`). Section 3a covers what it holds;
  the portal's own privacy notice at `https://app.terakota.io/privacy`
  covers the portal in full and says the same things this notice says.

## 3a. The terakota account (production QuickBooks connections, and the control panel)

**Who this applies to.** People who hold a terakota account. You get one in
one of three ways, each your choice: by signing up at `app.terakota.io`; by
connecting production QuickBooks Online through our connect service, under
our registered Intuit application (from terakota v1.4.0); or by running
`terakota login` (from terakota `v1.8.0`), which signs the binary in to an
account through our sign-in host in your browser — if there is no account
yet or its terms acceptance is not current, the binary sends you to the
browser to finish there; it never creates an account by itself, and a
signed-out or unlinked refusal never asks you to. AppFolio, Dialpad, local
reconciliation, receipts, `verify-receipts`, and Intuit sandbox companies
connected under your **own** registered Intuit application need no account,
and none of this section applies to them. If you never do any of the three,
no account of yours exists.

**Why the account exists.** Four reasons, and no others. Three belong to the
connect service: so we can notify you if it suffers a security incident; so
abuse of our shared Intuit application can be attributed to an account and
its refresh cut off (cutting refresh is the abuse control — it takes effect
within one access-token lifetime); and so we can reach you about security
matters affecting the connection. The fourth is the control panel: so that
seeing and changing a hosted tenant's delivery routing — in the browser or
through the terakota binary — is tied to a member an operator has switched
on, and every change is attributable to an account. Accounts are never
joined to any marketing list, and we do not use them to sell you anything.

**Account data.** Your email address and authentication details are held by our
sign-in provider, Auth0 (an Okta product, US tenant). We never store your
password. Our own control store keeps your account email, your Auth0 subject
identifier, digests of session tokens (never the raw tokens), your workspace
memberships and entitlements — including whether the control panel is
switched on for your account, a flag only an operator can set, and setting
or clearing it is one of the audited grant events — the version of the
Portal Account Terms you accepted and when, and an append-only audit log of
account, grant, and login events. The sign-in tokens `terakota login`
obtains are not stored by us anywhere: each control-plane call presents the
short-lived access token, we verify it and mint no session. Auth0 keeps the
sign-in grant records for the binary's client (the nearest thing to a device
list); see *Sign-in tokens on your machine* below.

**Connection record — the complete list.** For a production QuickBooks
connection made through our connect service, we store exactly these eleven
fields and nothing else:

1. portal account id
2. connection id
3. provider (which vendor the connection is for)
4. device-key thumbprint
5. client id
6. keyed HMAC of the normalized realm id
7. status
8. scope
9. created, last-refresh and revocation timestamps
10. token-generation HMAC
11. revocation reason

Three things are deliberately absent: **no raw realm id, no company name, no
token material.** This is a closed set, not a starting point — adding a field
requires amending the architecture decision that fixes it, and the published
notices change first.

**Connection attempts (flights).** Starting a connection mints a single-use row
that expires on a short timer (10 minutes by default). It records which vendor
the attempt is for, the state of the attempt, a verifier encrypted at rest, the
two public keys your install registered for that one attempt, and the loopback
address on your machine the result returns to. It never holds an authorization
code and never holds a token. Minting one requires a signed-in portal session.

**Token handling.** Token material transits the broker and is never at rest
there. Tokens are returned to your install as a capsule encrypted to a key only
that machine holds, signed by the broker, and bound to that one authorization
attempt. They are stored only on your machine. There is no plaintext OAuth token
and no broker-decryptable OAuth token at rest anywhere in our systems.

**Audited events.** Three connect events join the account audit log:
`qbo_connect`, `qbo_revoke`, and `qbo_refresh_denied`. They record that the
event happened, to which account and connection, and when.

**The control panel — what we record (from portal revision `2026-09-07`;
from terakota `v1.8.0` for the binary).** Every change made to a tenant's
delivery routing through the panel (and, from the release that ships the
binary's change commands, through the binary) — a subscription added or
removed, a held delivery replayed, a catch-all replay, and — by an operator,
for now — a topic registered or deleted — writes one append-only row in our
engine store: when it happened, which account (your account id — an internal
identifier, not your email), which tenant, which action, a digest of the
routing before and after, and the database login the change ran under. No
event content, no inputs beyond the action itself, and — for the binary —
never the `--intent` text you typed. From terakota `v1.8.0`, each
control-plane read and each `events-tail` poll is recorded too, but
separately and more briefly: one line in a read log in our control store —
your account id, the tenant, which read, and the time, with no inputs and no
results — and nothing at all for the command that only asks who you are
signed in as. If a read's line cannot be written, the read still completes
and the failure is noted in our application log — the tenant, the command
and the error, never your account id; that log lives on our host for a
bounded period and holds no identifier of yours. While a tail runs that is
one line every few seconds, which is a record of when your machine was
polling. We delete read-log lines older than 90 days, and closing your
account deletes yours in the same step that clears your email and display
name. The change rows above are different: they are append-only and have no
automatic expiry today. For each event delivered on a tenant's hosted spine
we also keep one index row — its sequence, topic, entity id, event id, the
delivery's message id, and when it was received and delivered; the table has
no payload column, so it cannot hold an event's content, and rows older than
90 days are removed on the operator's retention run. Neither the panel nor
the binary can see an ingest token, a quarantined delivery's body or
signature, a dead letter's payload, or a destination's address or secret;
dead-letter error text is shown with addresses masked. Other members of the
same tenant can see, on the panel's audit view, that a member account made a
change — the action, the time and the digest, not which account.

**Sign-in tokens on your machine (from terakota `v1.8.0`).**
`terakota login` leaves two tokens on your machine and nowhere of ours: a
refresh token that rotates on every use and an access token that expires
within an hour. They live in your operating system's keychain, or in
terakota's encrypted-file keystore if you opted into that fallback, beside
your vendor credentials and sealed the same way (on the file backend the
account entry has a passphrase of its own). `terakota logout` deletes both
from your machine and asks our sign-in provider to revoke the refresh token;
if the provider cannot be reached, the local deletion still happens and the
refresh token runs out on the provider's clocks below. If a refresh token
that has already been rotated is presented again — outside a ten-second
window that absorbs a retried request — our sign-in provider revokes the
whole family of tokens issued from it. A refresh token also expires on the
provider's clocks — 90 days after sign-in, or 30 days without use — after
which you sign in again. An access token already issued keeps working until
it expires, an hour at most.

**The link.** `terakota link` writes the tenant's name, the control plane's
address, and the time into that company's profile on your machine. The
membership check is one `platform-status` read, so it leaves the one
read-log line every read leaves (kept 90 days); beyond that nothing per link
is stored with us, and your local company id never leaves your machine.
`terakota unlink` removes it and contacts nothing.

**Logs and metrics for this surface.** Bounded outcome classes and aggregate
counts. No URL queries, no request or response bodies, no authorization codes,
no tokens, no state values, no capsules, and no realm ids appear in any log,
trace, or metric — including at the hosting edge.

**Retention and deletion.** Account data lives for the life of the account. There is
no self-serve close button: write to contact@bilans.io — the contact address on the
portal — and an operator runs the offboarding sequence. That sequence cuts your
access first — your workspace memberships and active sessions go, and a
control-plane access token already issued to the binary opens nothing from that
moment, because every control-plane call checks your membership live (it stays a
valid token until it expires, within an hour) — and the same day your identifying
data goes with it: your Auth0 user and your sign-in identity are deleted, and on the
account record itself your email address, display name, accepted-terms record and
verified-email flag are cleared and the connect and control-panel entitlements are
withdrawn. What is left that day is a de-identified record that cannot be signed in
to and cannot be granted access or a connect entitlement again. Records of
connections that have been **revoked** are kept for 90 days after revocation — that
window is what makes abuse attribution on our shared Intuit application possible —
and are then erased, together with the spent flight rows of those connections. The
de-identified account record is removed after that, in a second step: those retained
records reference it, and it cannot be removed while they do, which makes the 90-day
window a floor on its removal rather than a target. We compute the date it becomes
removable when we act on your request, and give you that date in our response.
Revoking the grant at Intuit itself is a separate step, and it is one we perform on
request: running `terakota qbo disconnect --company <id>` asks the broker to revoke
with Intuit, and removing the app in your Intuit account stays available to you at
any time. Closing the account does not fire that upstream revocation on its own; it
stops renewals, which ends the connection within one access-token lifetime. The
audit log is append-only for integrity: rather than deleting rows, we replace the
identifiers in them with a tombstone, keeping the event and dropping the person.
Audit entries are retained for 365 days. Deletions reach the next operator backup
rotation; for an erasure request we force a fresh backup rather than waiting.
Backups taken before an erasure can retain copies of the erased records until they
age out of rotation and are destroyed; backups are held by the operator alone and
are never used to serve traffic, and if a backup is ever restored, the erasure is
re-run against the restored data.

The control-plane audit rows described above live in our engine store and
are append-only; they carry your account id, not your email, and closure
does not rewrite them. Once your account record is de-identified, those rows
point at an identifier that no longer resolves to a person in our systems.
The read log described above is not in that group: we delete its lines after
90 days, and yours go with your identifiers when your account record is
de-identified.

**Who else touches this surface.** Fly.io hosts the broker, the portal, and the
Postgres control store (US, iad). Auth0/Okta handles identity. Cloudflare
answers DNS for `oauth.terakota.io` and `app.terakota.io` and serves nothing on
them. The full register, with what each one touches, is published with the
portal privacy notice.

**Intuit is not one of our sub-processors.** Intuit is your own vendor and an
independent controller of your QuickBooks data. Your reads run from your machine
to Intuit directly; the broker never terminates business-data traffic and never
proxies a vendor data API. What passes between us and Intuit is the credential
exchange itself — the code-for-token exchange, each renewal, and revocation —
and nothing else.

## 4. Correspondence you send us

If you email us (security reports, legal or privacy requests, support
questions), we receive and keep that correspondence — including your address and
anything you attach — for as long as needed to handle the matter and for our
records (2 years default, longer where law or an ongoing issue requires). Email
is processed by our provider (Google Workspace). **Do not send credentials,
tokens, or production financial data in email**; redact reports accordingly (the
Security Advisory & Support Policy explains how to report vulnerabilities
safely).

## 5. What we never do

We do not sell or share personal information (as those terms are defined under
the California Consumer Privacy Act). We do not buy data about you, track you
across sites, profile you, or use advertising technology.

Your business data never reaches us. Not a ledger line, not a query, not a
result — there is none of it in our systems to breach, disclose, or be compelled
to produce, and that stays true for production QuickBooks connections, whose
reads run machine-to-Intuit directly.

We will not tell you there is nothing else to breach. From terakota v1.4.0
there is: production QuickBooks credential material transits our broker, and
the connection metadata in Section 3a rests in our portal store. From portal
revision `2026-09-07` there is also the control panel: the routing of the
hosted tenants you belong to, the audit rows your changes and calls leave,
and the delivery-event index — metadata, never an event's content — rest in
our engine store, reachable by the panel through one database role that sees
only your tenant's rows. That is a real attack surface, it is why the
account exists, and it is why we would have somewhere to send a breach
notification. The Security Advisory & Support Policy
covers how we handle and disclose incidents on that surface.

## 6. Your rights, changes & contact

We honor privacy rights available to you under applicable law (access,
correction, deletion, portability, and others where they apply). What we hold
about you is short: correspondence, and — if you hold a terakota account — the
account, any connection records, the control-plane audit rows your actions
produced, and the 90-day read log of your control-plane reads, as listed in
Section 3a. Many requests will still find nothing retained, but every request
gets a real answer: write to contact@bilans.io and we will verify, respond
within the timeline applicable law sets (default: 30 days), and explain any
denial. Erasure runs in two steps. The day we act on your verified request
your identifying data goes — your Auth0 user, your sign-in identity, your
sessions and memberships, and, on the account record, your email address,
display name, accepted-terms record and verified-email flag — leaving a
de-identified record that cannot be signed in to and cannot be granted access
again. The record itself is removed in a second step, after the 90-day window
for revoked connection records in Section 3a has closed and those records and
their spent flight rows have been erased: they reference the record, and it
cannot be removed while they do, so that step falls after the response
deadline rather than inside it. We give you the date it becomes removable in
our response; the append-only audit log is tombstoned rather than rewritten,
as Section 3a describes.

[Change log:
v1.3 — the account gains a second purpose, our control panel, and the
software an optional bridge to it (from terakota `v1.8.0`). The lede adds
the third mode; §1's enumerated closed set of calls to us grows from two
classes to three, names our sign-in host and `app.terakota.io/api/v1`, and
every command that may call them, each poll of a running tail included; the
account-free "no call at all" sentence is conditioned on not having signed
in; a new §1 paragraph states what a control-plane call sends and receives.
§3a is retitled, names the three ways an account comes to exist, adds the
fourth reason it exists, the control-panel entitlement, three new blocks
(what the control panel records — matched byte for byte to the portal notice
— the sign-in tokens on your machine, and the link), the withdrawal of the
control-panel entitlement at closure, and the fact that engine-store audit
rows are not rewritten by erasure. §3 names the control panel on
`app.terakota.io`; §5 names the new breach surface; §6 names the audit rows
among what we hold. Nothing about AppFolio, Dialpad, the connect service,
the eleven-field connection record, or the retention windows changes.
Revision 2026-08-31 (terms-pack v1.2) — Seam Check at `check.terakota.io` is
retired, and the two places this notice pointed at it go with it: the lede no
longer names it in the surfaces this notice covers, and Section 3 drops the
carve-out sending you to the portal privacy notice for it. Neither place ever
described processing of ours — both only said where to look — so this removes
a pointer, not a disclosure: no data category changes, no retention window
changes, and no surface we still run loses its description. The surfaces this
notice covers are now downloads, the connect service, the website, and
correspondence.
Revision 2026-08-28 (terms-pack v1.2) — the published erasure wording now
matches what erasure does. Closure and an erasure request complete in two
steps, not one: the day we act on a verified request the identifying data goes
(the Auth0 user, the sign-in identity, sessions and memberships, and the email
address, display name, accepted-terms record and verified-email flag on the
account record), and the account record itself is removed only after the
90-day window for revoked connection records has closed, because those
retained records reference it and it cannot be removed while they do. Section
3a now states both steps, the de-identified state between them, and that we
give you the date the record becomes removable; Section 6 separates the
response deadline (default: 30 days), which is unchanged, from the removal of
the record, which falls after it. This corrects a description, not a practice:
no new data category, no new retention, and the 90-day and 365-day windows are
unchanged.
Revision 2026-08-17 (terms-pack v1.2) — item 9 of the connection-record
enumeration now names the revocation timestamp (`revoked_at`) beside the
created and last-refresh timestamps; the field was added so the 90-day
erasure of revoked connection records published in Account Terms Section 9.3
is executable. No new data category — a stored-field disclosure
clarification, and the enumeration stays closed at eleven items.
v1.2 — Dialpad is named in the enumerations that were already true of it: the
lede's account-free mode, the "makes no call to any host of ours" list, the
never-transmitted list (§1), and the who-this-applies-to carve-out (§3a). What
we hold does not change. Reads with a Dialpad API key you supply run from your
machine to Dialpad directly, add no host of ours to any flow, and put nothing
new in our systems — the enumerated closed set of network calls to us (§1),
§3a, and the breach surface named in §5 are all untouched.
v1.1 — production QuickBooks connections through our hosted connect service
(from terakota v1.4.0). The lede is split by mode: the software still sends us
nothing for AppFolio, local use and `verify-receipts`, but production QuickBooks
runs through a service we operate and needs a free account. Network calls to us
are now an enumerated closed set of two, including the automatic token refresh
(§1). Token material for production connections is disclosed as transiting the
broker, sealed to the requesting machine, nothing at rest (§1, §3a). New §3a
enumerates the account data, the eleven-field connection record, flight rows,
the audited events, the telemetry fence, purposes, retention and deletion, the
marketing fence, and why Intuit is not a sub-processor. The surface inventory
gains `oauth.terakota.io` and `app.terakota.io` (§3). The "nothing of it for us
to breach" sentence is corrected — there is now a breach surface, named (§5).
The rights section names what we actually hold (§6).
v1.0 — first published version.]
