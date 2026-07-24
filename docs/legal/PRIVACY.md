# terakota Privacy Notice

Version 1.0 — Effective 2026-07-24 — Bilans Solutions LLC

The short version: **the terakota software sends us nothing, requires no
account, and we run no service it depends on.** This notice describes the few
places our infrastructure can observe anything at all: downloads, the website,
and correspondence you send us.

## 1. The software: nothing automatic, by architecture

`terakota` and `verify-receipts` run on your machine. They need no terakota
account and no sign-in with us; they have no telemetry, no crash reporting, no
analytics, and no network calls to us — none, not even optional ones, in this
version. Your credentials, OAuth tokens, queries, query results, and receipt
chains are stored and processed only on your machines; they are never
transmitted to us and we have no ability to access them. The software's only
network connections are to the systems you connect it to (AppFolio, Intuit)
using your credentials. Where the software stores local data (keystore, receipt
chains, config) and how to delete it is documented in the release repository —
deletion is yours to perform; we hold no copy.

If a future release adds any optional feature that contacts us (such as a
user-invoked version/advisory check), this notice will be updated first: the
effective date above will move, a change note will be added below, and the
change will be announced on the release repository before it takes effect.

## 2. Downloads

Releases are distributed via GitHub. When you download, GitHub processes your
request under its own privacy policy; we see only the aggregate download counts
GitHub exposes to repository owners — no identities, no IP addresses.

## 3. The website

terakota.io is a static informational site hosted on Cloudflare Pages. It uses
cookieless, aggregate analytics (Cloudflare Web Analytics) — no cookies, no
cross-site tracking, no user identification. Hosting-level logs are handled by
Cloudflare under its own policies; we retain no server logs of our own. No
accounts exist on the website and no forms collect personal information.

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
across sites, profile you, or use advertising technology. Your business data
never reaches us — there is nothing of it for us to breach, disclose, or be
compelled to produce.

## 6. Your rights, changes & contact

We honor privacy rights available to you under applicable law (access,
correction, deletion, portability, and others where they apply). Because we hold
little — correspondence, and nothing else tied to you — many requests will find
nothing retained, but every request gets a real answer: write to
privacy@terakota.io and we will verify, respond within the timeline applicable
law sets (default: 30 days), and explain any denial.

[Change log: v1.0 — first published version.]
