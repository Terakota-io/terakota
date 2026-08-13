# terakota Privacy Notice

Version 1.1 — Effective 2026-08-13 — Bilans Solutions LLC

The short version, split by what you connect:

- **AppFolio, local reconciliation, receipts, and `verify-receipts`:** the
  terakota software sends us nothing and requires no account.
- **Production QuickBooks connections (from terakota v1.4.0):** these run
  through a connect service we operate, and they require a free terakota
  account.

This notice describes every place our infrastructure can observe anything at
all: downloads, the connect service (the broker at `oauth.terakota.io` and the
portal at `app.terakota.io`), the website, and correspondence you send us — plus
Seam Check at `check.terakota.io`, which the portal privacy notice covers.

## 1. The software: what it sends, and to whom

`terakota` and `verify-receipts` run on your machine. They have no telemetry, no
crash reporting, and no analytics — none, in any version, in any mode. There is
no background reporting of any kind.

The calls the software makes to us are a closed, enumerated set of exactly two
classes:

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

If you use AppFolio, local reconciliation, receipts, `verify-receipts`, or an
Intuit sandbox company under your own registered Intuit application, the
software makes no call to any host of ours at all, and needs no account.

**What is never transmitted to us, in any mode:** your AppFolio credentials,
your queries, your query results, and your receipt chains. They are stored and
processed only on your machines, we hold no copy of them, and no interface of
ours can reach them.

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

Where the software stores local data (keystore, receipt chains, config) and how
to delete it is documented in the release repository — deletion is yours to
perform; we hold no copy.

If a future release adds any feature that contacts us beyond the two calls
enumerated above, this notice will be updated first: the effective date above
will move, a change note will be added below, and the change will be announced
on the release repository before it takes effect.

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
- **`app.terakota.io` — the portal.** Sign-in gated (Auth0). This is where a
  terakota account lives and where a connection is started. Section 3a covers
  what it holds; the portal's own privacy notice at
  `https://app.terakota.io/privacy` covers the portal in full, and — from its
  next revision, published with this one — says the same things this notice
  says.

`check.terakota.io` (Seam Check, the free one-shot reconciliation) is covered by
the portal privacy notice, not here.

## 3a. The terakota account (production QuickBooks connections)

**Who this applies to.** Only people who connect production QuickBooks Online
through our connect service, under our registered Intuit application (from
terakota v1.4.0). AppFolio, local reconciliation, receipts, `verify-receipts`,
and Intuit sandbox companies connected under your **own** registered Intuit
application need no account, and none of this section applies to them. If you
never connect production QuickBooks through our service, no account of yours
exists.

**Why the account exists.** Three reasons, and no others: so we can notify you
if the connect service suffers a security incident; so abuse of our shared
Intuit application can be attributed to an account and its refresh cut off
(cutting refresh is the abuse control — it takes effect within one access-token
lifetime); and so we can reach you about security matters affecting the
connection. Connect-service accounts are never joined to any marketing list, and
we do not use them to sell you anything.

**Account data.** Your email address and authentication details are held by our
sign-in provider, Auth0 (an Okta product, US tenant). We never store your
password. Our own control store keeps your account email, your Auth0 subject
identifier, digests of session tokens (never the raw tokens), your workspace
memberships and entitlements, the version of the Portal Account Terms you
accepted and when, and an append-only audit log of account, grant, and login
events.

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
9. created and last-refresh timestamps
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

**Logs and metrics for this surface.** Bounded outcome classes and aggregate
counts. No URL queries, no request or response bodies, no authorization codes,
no tokens, no state values, no capsules, and no realm ids appear in any log,
trace, or metric — including at the hosting edge.

**Retention and deletion.** Account data lives for the life of the account.
There is no self-serve close button: write to contact@bilans.io — the contact
address on the portal — and an operator runs the offboarding sequence. That
sequence cuts your access, then removes the account record, the
Auth0 user, live connection records, and any flight rows. Records of connections
that have been **revoked** are kept for 90 days after revocation — that window
is what makes abuse attribution on our shared Intuit application possible — and
are then erased. Revoking the grant at Intuit itself is a separate step, and it
is one we perform on request: running `terakota qbo disconnect --company <id>`
asks the broker to revoke with Intuit, and removing the app in your Intuit
account stays available to you at any time. Closing the account does not fire
that upstream revocation on its own; it stops renewals, which ends the
connection within one access-token lifetime. The audit log is append-only for
integrity: rather than deleting rows, we replace the identifiers in them with a
tombstone, keeping the event and dropping the person. Audit entries are retained
for 365 days. Deletions reach the next operator backup rotation; for an erasure
request we force a fresh backup rather than waiting. Backups taken before an
erasure can retain copies of the erased records until they age out of rotation
and are destroyed; backups are held by the operator alone and are never used to
serve traffic, and if a backup is ever restored, the erasure is re-run against
the restored data.

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

We will not tell you there is nothing else to breach. From terakota v1.4.0 there
is: production QuickBooks credential material transits our broker, and the
connection metadata in Section 3a rests in our portal store. That is a real
attack surface, it is why the account exists, and it is why we would have
somewhere to send a breach notification. The Security Advisory & Support Policy
covers how we handle and disclose incidents on that surface.

## 6. Your rights, changes & contact

We honor privacy rights available to you under applicable law (access,
correction, deletion, portability, and others where they apply). What we hold
about you is short: correspondence, and — if you have connected production
QuickBooks through our connect service — your account and its connection records
as listed in Section 3a. Many requests will still find nothing retained, but
every request gets a real answer: write to contact@bilans.io and we will
verify, respond within the timeline applicable law sets (default: 30 days), and
explain any denial. Erasure reaches the account record, the Auth0 user,
connection records, and flight rows; the append-only audit log is tombstoned
rather than rewritten, as Section 3a describes.

[Change log:
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
