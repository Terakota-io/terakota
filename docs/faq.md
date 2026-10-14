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

**What is the evidence pack?** From v1.6.0, `terakota evidence --company <id> --out
pack.zip` composes a company's local receipt chain into one self-contained set you
can hand to someone: `receipts.jsonl`, a scannable `receipts.csv`, a printable
`timeline.html` — one plain-English row per execution, showing what terakota
executed on the agent's behalf, from which system, with the caller's stated intent
labeled as caller-supplied and unverified — plus `manifest.json` and a `VERIFY.md`
walkthrough. It holds no vendor record bodies, and composition is verify-gated: a
chain that fails verification writes nothing. The recipient re-verifies it offline
with `verify-receipts`, no terakota process running. The evidence class is
**artifact integrity**, and it inherits the limits above: a partial change breaks
the hash linkage and shows, while a wholesale replacement is only detectable
against a chain head you recorded separately — so write down `chain_head` from
`manifest.json` when you receive a pack. The pack's `VERIFY.md` states all of this
in full.

**What does `terakota reconcile` prove?** Less than the word "prove" suggests. From
v1.7.0, `terakota reconcile --company <id> --from <date> --to <date>` pulls both
connected sources over a closed posting-date window, runs the deterministic matcher over
the journal entries it finds, and records the links and the verdict on your local chain;
every pull is an ordinary receipted read, so the reads it made sit on the chain beside
the verdict. What offline verification establishes is narrower: that the verdict
**record** is intact and self-consistent — its inputs manifest still hashes to its own
`input_hash`, its window and its coverage claim agree with that manifest, and its lineage
fields are well-formed and correctly ordered against their neighbours. `verify-receipts`
does not re-read the sources, re-run the matcher, recompute the link-set hash from the
link records, or tie the inputs manifest to the `read_result` rows beside it — so a pass
does **not** prove the verdict was derived from those reads, and it does not establish
that your books are correct. [verify.md](verify.md) states the same limit at length.

Its coverage is stated rather than assumed: any partial leg makes the run partial and no
`missing` is asserted, a failed page fails the whole run instead of minting a verdict off
half a window, and two classes — `backdated` and `edited_after_rekey` — are declared
**not detectable** on this shape rather than reported as zero, because they need
persisted state that does not exist on your machine.
Every correlation key reads un-attested: `derivation: key` carries no proven uniqueness.
Reconcile is verified on synthetic two-source journals; no customer's books have been
reconciled with it yet. [reconcile.md](reconcile.md) carries the full statement,
including what a link record discloses when it leaves your machine.

**Can an agent confirm a link?** No. `links-confirm` and `links-revoke` require an
interactive controlling terminal and have no MCP counterpart — the server exposes no
confirm tool at all — so the review verbs are not reachable from a transport terakota
governs. That is a bar rather than a proof, and the refusal says so itself: a pty can be
faked by anything with shell access, so what the fence buys is that an agent cannot
confirm a link by accident on its way past, and the receipt each verb writes names the
review **surface** (`human:cli`), never an actor.

**Can I change routing from the CLI?** From v1.10.0, yes, for a company you have linked
to a hosted Terakota tenant. `terakota platform-subscribe` creates or updates one routing
row, `platform-unsubscribe` removes one, `platform-replay` requests the replay of one
quarantined capture, and `platform-catchall-replay` re-routes the preserved events whose
topic is now registered and subscribed. All four are CLI verbs only: no MCP row and no
agent tool. Each runs the same ceremony. It prints the act it is about to perform, naming
the tenant, the company, the verb and its inputs, and proceeds only when you type the
tenant slug back exactly, within 120 seconds; a wrong word, a bare newline or silence
cancels without dialling anything. Unlike the reads, each change is a receipted act: a
`control_intent` lands on the company's chain before the control plane is dialled and a
`control_act` once the attempt ends, whether a readable answer came back or not. The act
records `ok` with the effect and the audit row the control plane wrote, `refused` with the
control plane's own reason, or `error` when nothing readable arrived at all: a dial that
failed, a timeout, a body that is not an envelope, or a class the binary does not know.
Every act is appended on a context detached from cancellation, so interrupting the command
after it dialled still writes one. An intent left with no act therefore means the process
died between the two appends, or the append to the chain itself failed. `terakota
evidence` folds the pair into one timeline row: the verb and the tenant in the title, the
inputs as one fact, the outcome as the status, and your stated intent beneath it, labelled
caller-supplied and unverified as always.

**Why does a change command refuse in an agent or a script?** Because stdin is not an
interactive controlling terminal and the four change commands require one. The class is
`no_controlling_terminal`; the refusal fires before a profile is read, a chain is opened
or a keystore is touched, and there is deliberately no flag, environment variable or
automation path that bypasses it. It is the same fence `links-confirm` and `links-revoke`
carry, with the same limit stated in the refusal itself: a pty can be faked by anything
with shell access, so it is a bar rather than a proof. What it buys is that an agent
cannot re-route a tenant's stream on its way past. The reads need no terminal and are
unaffected, so an agent can still see what the routing is and propose a change; running
the change is a human act at a terminal, and the receipt pair records the surface as
`human:cli`.

**What does `mutation_refused` mean?** That the control plane read your change, was
entitled to act on it, and declined on the merits. The detail names its own reason token,
and that token is the only thing the control plane says which reaches your terminal. The
list is closed: `absent`, `already_requested`, `already_replayed`, `not_routable`,
`empty_registry`, `destination_cap`, `chartered`, `destination_foreign`,
`binding_refused`, `queue_profile`, `source_unbound`. Nothing was written on the control
plane and no audit row exists for it, and the `control_act` on your chain records
`refused` with that token as its `outcome_class`. Read the reason, correct the inputs,
re-run. A token outside that list is not treated as a refusal at all: an answer the
binary cannot read is an outcome it never observed, so the act records `error` with class
`control_plane_unavailable`, and that is the one case where whether the change landed is
genuinely not known here. Re-running the same command is how you find out, because each
change states the routing you want, so a re-run either applies it or reports what already
stands.

**What is `chain_held`?** Another process already holds that company's receipt chain,
usually a running MCP extension or a second `terakota` working on the same company. A
change opens and locks the chain because it writes to it, which the `platform-*` reads
never do, so a change is the one linked verb that will not run beside a live
`terakota mcp` on that company. Nothing was changed, nothing was dialled, and the
passphrase was never asked for. Close the other process, or point the change at a
different `--home`, and re-run.

**What is the mirror transport?** From v1.11.0, a company's AppFolio tools can read a
Postgres mirror of its AppFolio data instead of the AppFolio API. It is for a company that
already has its AppFolio data copied, by a separate sync engine, into a Postgres database
it controls. terakota reads that database from your machine with a read-only role; no
service of ours sits in the path. The transport is chosen per company and the API stays the
default, so a company that never binds a mirror reads exactly as before. The same 17
collections answer through the same `appfolio_*_get` and `appfolio_*_list` tools, with the
API's page sizes; no tool was added or removed. Read-only still holds: the session is
read-only, each read runs in one read-only, repeatable-read transaction (so a page and its
child rows come from one snapshot), and the bind refuses a role that can write the tables
it reads.

**How do I bind a mirror?**

    terakota credentials set --company mybooks --transport mirror --mirror-env production

`--mirror-env` is required and says what the database holds: `dev` or `production`. The
database URL is never an argument. The command asks for it at a no-echo `Database URL`
prompt, or reads it with `--url-fd <n>` from an open file descriptor, which is the way in
without a terminal; the descriptor is read to its end and must carry the URL alone on one
line. The URL accepts three options, `sslmode` (`require` or `verify-full`), `sslrootcert`
(an absolute path, with `verify-full` only) and `connect_timeout`, and refuses any other.
Port 6543, a transaction-mode pooler, is refused: use a session-mode port.

Before it stores anything, the bind connects read-only and checks the database, reporting
every failure at once:

- the role is not a superuser, owns none of the tables terakota reads, and cannot write
  them;
- every table and column terakota reads exists;
- no table carries a tenant-key column, because one mirror must hold exactly one AppFolio
  database.

If the company also has AppFolio API credentials, the bind then looks the mirror's newest
property up over the API: found prints `same_database: proven`, while a miss, an empty
mirror or an API it cannot read refuses the bind. Without API credentials it prints
`same_database: unproven`.

**What does the bind print, and what is stored where?** On success, one line:

    stored mirror binding for company "mybooks": <user>@<host>:<port>/<dbname> [production]; AppFolio reads now use the mirror

with ` tls: unverified` after the environment unless the URL set `sslmode=verify-full`.
The identity names the user, host, port and database, never the password. Then come the
sync engine's own state labels and names, so you can check by eye where freshness is read
from, and `same_database`. A role wider than terakota needs adds a line on stderr. A
failure never prints the URL or the password.

Only the password is sealed: in your keystore (the OS keychain by default) under
`appfolio-mirror/<id>`, beside the company's other entries, so `keystore migrate` moves it
with them. The rest of the binding holds no secret and goes in the company's entry in
`profiles.json`: user, host, port, database and schema, the environment and connection
settings, and what the bind found (role scope, `same_database`, the ledger-detail floor and
when the binding was pinned).

Running the bind again with the same user, host, port, database, schema and environment
rotates the password. Anything else, or weaker TLS (dropping `verify-full`, or another root
certificate), is refused unless you pass `--replace`, which prints the old and the new
identity and how many reconcile runs the company has recorded. A `--replace` needs the
company's chain free, so close a running MCP server first. It does not end a list walk in
progress either: after a rebind, list again without the cursor.

**How do I switch back, or check a binding?**

    terakota company transport --company mybooks               # prints api or mirror
    terakota company transport --company mybooks --set api     # back to the AppFolio API
    terakota company transport --company mybooks --set mirror  # back to the mirror
    terakota company transport --company mybooks --check       # re-run the mirror handshake

`--set` switches only to a transport that already holds stored credentials, asks for no
passphrase, and leaves the other transport's credentials stored. A running MCP server keeps
the transport and binding it opened with until it restarts. A plain `credentials set` on a
mirror-bound company stores the API pair, leaves the transport on `mirror`, and says so.

`--check` re-runs the handshake on the bound mirror and prints the identity line, the
engine's labels, `tables N/M`, `gl_details from <date>`, `role_scope: narrow|wide`,
`same_database` and `tls: verified|unverified`, then one line per collection: when its
events were last applied and received, how many are pending, and its notes. It refuses a
company on the API transport, and exits `1` when a production binding's role has become
wide. `--set` and `--check` together are refused.

**What does `coverage.mirror` tell me?** Every AppFolio answer for a mirror-bound company
carries it, errors included once the read has started; an API answer never does.

- `last_event_applied_at`: when the collection's newest change was applied to the mirror.
  Charges take it from the engine's last charges job (or the table's newest update),
  gl_details from the engine's ledger sync cursor, and work orders from the table's newest
  update when the queue has none.
- `last_event_received_at`: the newest event the sync engine received, one value for every
  collection.
- `pending_events` and `failed_events`: the collection's changes still queued, and those
  that failed to apply. Always present, zero included.
- `fields_absent` and `fields_lossy`: the tool's declared fields the mirror cannot carry,
  or carries with a loss.
- `notes`: tokens from a closed list of eleven.
  - About the collection: `deletes-not-mirrored` (every collection), `charges-manual-job`
    (charges refresh only when the engine's charges job is run by hand),
    `sync-time-timestamp` (gl_details and rental applications carry the engine's sync time,
    not AppFolio's), `memberships-incomplete` (property group memberships) and
    `hidden-by-is-active` (owners: hidden is judged by an active flag).
  - About freshness: `liveness-unproven`, below.
  - About the read: `below-mirror-floor`, `absent-in-mirror`, `deleted-flag-unverified`.
  - About the binding: `tls-unverified`, `role-scope-wide`.

A time that cannot be proven is left out, never guessed.

**What do the mirror's `next_action` hints mean?** Four of them are about the mirror
itself, and none rides an error:

- **Stale:** the collection's freshness is not proven current, so its records may lag
  AppFolio. For charges, gl_details and work orders that means the applied time is missing
  or over an hour old; for the rest, the sync engine has received no event at all for
  over an hour, or `liveness-unproven` rides.
- **Silent** (`liveness-unproven`): the sync engine has no heartbeat, so a quiet queue looks
  the same as a stopped one. The note rides when a collection has no applied time, has
  received no event of its type in 24 hours, or its freshness could not be read. Outside
  charges, gl_details and work orders it brings the stale hint with it.
- **Failed events:** the hint gives the count of the collection's changes that failed to
  apply, and says to confirm over the API before acting on its records.
- **Found record:** a `_get` that finds the record says it was present as of
  `last_event_applied_at`, that deletions are not mirrored, and to confirm over the API
  before acting on its existence.

The way to the API is `terakota company transport --company <id> --set api`. It takes
effect on the next process start: the next CLI command, or the MCP server's restart.

**Why are counts over the mirror exact, and what does "as of" mean?** Over the mirror an
`--aggregate` runs as one SQL statement inside one snapshot: no page walk, no cursor, no
carried subtotal to stitch together. The aggregate block says `exact: true` when the
collection's applied time is known and the group cap did not cut the answer, and `as_of`
names that time, the collection's `last_event_applied_at`. Exact means exact over the rows
the mirror held, not over AppFolio now: a change the mirror has not applied yet (see
`pending_events` and `failed_events`) is not in the count. The API's 1,000-group cap still
holds: past it, the answer is the first 1,000 groups in key order, `partial` and without
`exact`, and `next_action` says to group by a coarser field or narrow the read. gl_details
debit and credit totals come from their own statement over the whole window, so they stay
exact under the cap.

**What does the mirror not carry?** Three things, each disclosed on the answer:

- **Deletions.** A record deleted in AppFolio stays in the mirror until the sync engine
  removes or flags it, and today the engine does not mirror deletions: the note
  `deletes-not-mirrored` rides every collection. Lists and aggregates leave flagged rows
  out. `_get` never answers `gone` over the mirror: a miss is a `gap`, noted
  `absent-in-mirror`, or `deleted-flag-unverified` when the mirror flags the row deleted.
- **Some fields.** `fields_absent` names declared fields the mirror has no column for, and
  `fields_lossy` the ones it carries with a loss, such as a date without its time or an
  enum value the engine could not store.
- **Ledger detail before the floor.** The mirror holds gl_details from its earliest
  posting date on, which `--check` prints as `gl_details from <date>`. A window wholly
  below that floor, or a mirror with no ledger detail, answers `error` with class
  `source_rejected` and a detail naming the floor, or saying there is none. A window that
  crosses it is answered from the floor up as `partial`: `coverage.suggested_window` names
  the part below, and `next_action` says to read it over the API. Over the mirror,
  gl_details also refuses `--updated-on-from` / `--updated-on-to` and
  `--group-by Postings`.

**Why is a CLI call over the mirror slower than an MCP call?** A process opens the mirror
once: connect, TLS, sign-in, then the handshake that checks the role and the tables. Each
`terakota` command is its own process, so each one pays that before its read: about three
seconds per command in our measurements against a remote pooler. `--check` also reads
freshness for all 17 collections and takes longer. The MCP server opens the mirror on its
first AppFolio read and keeps it for the session, so it pays once. Every answer still
reads its collection's freshness.

**What does a production binding need?** `--mirror-env production` requires
`sslmode=verify-full` with `sslrootcert`, the absolute path of the root certificate the
server is checked against, and a narrow read-only role: one that reads the `appfolio`
schema only. A role that can also read the sync engine's other schemas is refused at bind
time, by `--check` (exit `1`) and at read time. A `dev` binding accepts `sslmode=require`
and a wider read-only role, and says so: the notes `tls-unverified` and `role-scope-wide`
ride every answer, `tls: unverified` follows the environment on its identity line, and a
wide role adds a line on stderr.

**What does a reconcile over the mirror record?** For a mirror-bound company,
`terakota reconcile` and the MCP `reconcile_start` read the AppFolio side over the mirror;
QuickBooks is read over its API as before. A binding that cannot open refuses the run; it
never falls back to the API. Each AppFolio leg is an ordinary receipted read carrying the
mirror's `adapter_version` and an `afm:` request id. The reconciliation receipt names the
input on its manifest: the AppFolio line carries `source_class: db_mirror` and
`auth_verdict: db_role_readonly`, and the QuickBooks line stays `api_sweep` and
`transport_auth`. Harvest tiles below the mirror's ledger-detail floor are not read, and a
tile that crosses it is read from the floor up; what lies below is recorded unswept under
the cause `source_floor`, the run is `partial`, and `next_action` counts those windows and
says to reconcile those dates over the API. The run
envelope carries one `coverage.mirror` for the collections the run read; from an MCP
session, `reconcile_status` carries it only from the server process that ran the run.

A run cannot switch transport halfway. The local run row records the transport the run
started over, and resuming it over the other one refuses before either source is read:
`run <id> started over the api transport and cannot resume over the mirror; start a new run`
(or the reverse). The row settles `failed` with `environment_moved`, and the next
reconcile of that window starts a new run. Rows written before v1.11.0 record no transport
and resume as they did. [verify.md](verify.md) covers what the verifier checks on these
records.

**Why did `registry_version` change?** Because v1.7.0 added the three composite tool rows
to the static manifest and closed each row's receipt-contract declaration into an enum,
and `registry_version` is a hash over the whole table — so it rehashes as a whole:
`32e19c69…` (v1.5.0 / v1.6.0) becomes
`52c6473b294269cfc5b9aab6fd68254cab78795e32e53ffe0277b84909340422`, with the registry
growing from 39 to 42 tools. Receipts minted by v1.7.0 carry the new value. It is not a
break — `verify-receipts` grades the chain, not the registry value, so a chain holding
receipts from both versions still verifies. List cursors are unaffected: a cursor binds
to its collection and cursor format, not to the registry, so a walk started on v1.6.0
continues on v1.7.0. Continuing a walk has its own rules, and the upgrade does not change
them: pass `--company` — required on every invocation, cursor or not — and the same
non-default `--home`, and pass no filter flag alongside `--cursor`. Only `--intent` may
ride with it.

It moved again in v1.11.0, from
`31e9e888e01f2e8ff4fc4d460710dd6c12fa72c0b906465f45dc669cb504ed26` (v1.8.0 to v1.10.0) to
`5e53bb7facb0031db77174221360acc168e1e96d718e7418394ac1d9055753dd`. Five declarations
changed, on the `_get` and `_list` rows of five collections, and nothing else: `bills`
gained `Reference`, `leases` gained `SignedOn`, and `recurring_charges.Amount`,
`tenants.CurrentRent` and `vendors.WorkOrderAdjustment` are now declared decimal strings
rather than numbers, which is what the API sends. These are the field lists `--help` and
the MCP tool descriptions print; API answers are unchanged and no tool was added or
removed. The same rule holds: the verifier compares no registry value, so a chain holding
receipts from both sides of the move still verifies.

**Does terakota phone home?** No telemetry, no analytics, no crash reporting —
none, not even optional ones, in any version. It calls a service of ours in two
cases only. The first is a production QuickBooks connection made through our
connect service (from v1.4.0): the authorization and the revocation you ask for,
plus the token renewal, which runs on its own whenever the access token nears
expiry (roughly hourly in active use). The second is optional and exists from
v1.8.0: `terakota login` signs the binary in through our sign-in host, and after
you link a company with `terakota link` the `platform-*` read commands and
`events-tail` read our control plane, and from v1.10.0 the four `platform-*` change
commands post to it. Every one of those calls is started by you. With no
production QuickBooks connection and no `terakota login` it contacts no host of
ours at all, and its network connections are to the systems you point it at
(AppFolio or a database mirror of it you bound, Intuit, Dialpad), using your
credentials.

**Does a change command send anything else?** No, and the traffic is short. The change
itself is one POST to the control plane carrying three things: the bearer `terakota login`
obtained, the tenant slug as a path segment, and that verb's own inputs as a single JSON
object. Two things can ride alongside it. If the access token has expired, the binary
renews it with the sign-in service first, and that request carries the refresh token and
nothing else, never the change, your inputs or your intent. And if the control plane
answers 401, the binary tries once more for a renewed token and re-sends the same POST
with it, but only when the token actually changed; the same bearer is never presented
twice. Never sent at all: the `--intent` text, which is echoed on your envelope and
recorded on the receipt pair and never leaves the machine; the local company id; the
invocation id; the dedup key; the receipt pair itself; anything from your vendor sources;
and anything else on your machine.

What comes back depends on how the change went. When it was written and audited, the
envelope carries one record, naming the effect, the `audit_seq` of the audit row on our
side and that row's two digests, with `replayed` and `left` counts on the catch-all sweep
alone, and the act on your chain records `ok`. When the control plane declines on the
merits, the envelope carries no record and names its reason token in the detail, no audit
row exists for it, and the act records `refused`. When nothing readable arrives, the class
is `control_plane_unavailable` and the act records `error`, which says the outcome was
never observed. Once the receipt pair exists the envelope carries `receipt_ref` whatever
the outcome. No answer of any kind carries a payload, an endpoint or a secret reference,
and the set of hosts the binary can dial at all is the one described under **Does terakota
phone home?** above; a change adds no host to it.

**What data can you (the makers) see?** None of your business data, in any mode.
Your AppFolio and Dialpad credentials (a mirror binding's password included), your
queries, your results, and your receipt chains stay on your machine — we hold no copy
and no interface of ours can reach them. Reads run from your machine to the vendor, or
to the database mirror you bound, directly, always.

If you connect a **production** QuickBooks company through our connect service
(from v1.4.0), we hold two things: your free terakota account, and an eleven-field
connection record — ids, a keyed hash of the realm, status, scope, timestamps, and
a revocation reason; metadata only, never your QuickBooks data, and no raw realm id
or company name. QuickBooks token material transits the broker during authorization
and renewal and comes back sealed to your machine; nothing we could read is at rest
on our side. AppFolio (over its API or a database mirror), Dialpad, local
reconciliation, `verify-receipts`, and sandbox QuickBooks under your own registered
Intuit application need no account and put no service of ours in the path. The full
field list is in [PRIVACY.md](legal/PRIVACY.md) §3a.

If you sign in with `terakota login` and link a company (from v1.8.0), we hold two
more things, both described in Privacy §3a and neither of them your data: a read-log
line for each control-plane read or `events-tail` poll — your account id, the tenant,
which read, the time; deleted after 90 days — and, for each routing change, whether it
was made in the browser panel or with a v1.10.0 change command, one append-only audit
row in our engine store.

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
under our registered Intuit application, gated by a free terakota account. From
v1.6.0 that ceremony runs on a back channel: the CLI registers the connection keys
with the connect service directly, the browser carries only an opaque handle, and
the CLI prints a pairing code to match against the consent page before you approve.
Starting a **new** production connection therefore needs v1.6.0 or later — the older
URL form is refused server-side — while refreshing an existing connection is
unaffected. AppFolio and Dialpad reads are unaffected too.

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
