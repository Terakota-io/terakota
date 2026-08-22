# `terakota reconcile`

One blocking command. It pulls both connected sources over a closed posting-date
window through ordinary receipted reads, runs the deterministic matcher over the
journal entries they release, records the links and the run's verdict on your local
chain, and prints one response envelope on stdout.

    terakota reconcile --company mybooks --from 2026-06-01 --to 2026-06-30 \
                       [--progress json|plain|off] [--intent "june close"] [--home <dir>]

Honest tier: reconcile is verified on synthetic two-source journals and on a read-only
live AppFolio rail; no customer's books have been reconciled with it yet.

## The window

`--from` / `--to` are the **declared correspondence**: a closed civil-day window on the
**posting** date, both bounds required and the upper day included. They are not the
update axis the two journal reads walk — a change-window population answers "what
moved", while a verdict claims a period.

## The config

The matching rules come from `<config>/companies/<id>/reconcile.yaml` and nowhere else.
There is **no `--config` flag**: the file is read from that path or the run is refused,
and the refusal names the exact path it looked at. terakota never writes the file for
you. Write it yourself, mode `0600`.

Four sections, all optional except the ones your books need:

    accounts:
      - code: "1000"
        name: Operating Bank
        primary: ["1000"]
        comparison: ["1000", "Operating Bank"]

    entities:
      - name: acme-property-1
        primary: ["acme-property-1"]
        comparison: ["acme-property-1"]

    matching:
      date_window_days: 3
      amount_tolerance_minor: 0
      missing_lag_days: 5
      backdate_grace_days: 7

    sensitive_fields:
      - payment_instrument

Unknown keys are **refused**, not dropped — a typo in a key name stops the run rather
than silently changing what the matcher does.

`matching.pick_policy` is **not admitted**, in any form. A file that sets it is refused
at load with a named error. No pick lane exists on this shape, and a picked link's lane
is not recoverable from the chain, so refusing the section is what makes "no fuzzy
matching" a bound rather than a default.

The file's sha256 **is** the `config_version` every link and every verdict carries.
Editing the file therefore moves the version, and the next run records a supersede for
every link the old config minted — with kind `reevaluate` for links a person had
confirmed, which are flagged for revalidation rather than silently carried forward.

## What a run touches

- **AppFolio** — the property, unit and GL-account reads that resolve names, then a
  per-property date-windowed `gl_details` harvest in ≤31-day tiles, then the
  transactions it found hydrated through the ≤50-id `journal_entries` batch.
- **QuickBooks** — one `TxnDate`-windowed journal walk on the posting axis.

Every one of those is an ordinary `read_intent` / `read_result` pair on your chain. Run
cost scales with the company's **property count** as well as the window length, because
the harvest fans out per property. The harvest carries its own declared fetch budget,
and a tile the budget cuts is recorded **unswept** — a coverage fact, never silence.

The property and unit reads ask for hidden records too (`include_hidden`, visible on the
receipt's `normalized_params`). A property left out of that list is one this run never
reads, and its QuickBooks counterparts would then read `missing` off a population nobody
looked at. If a reference read ever comes back from a filtered population anyway, the leg
is recorded partial by name rather than passed off as whole.

An ids batch is judged by the **ids it answered**, not by how many records came back: a
duplicate or a record nobody asked for would otherwise make the count add up over a set
with a hole in it.

## What the verdict says, and what it does not

Coverage governs what absence means. Any partial leg — a tripped bound, a budget-cut
tile, or an ids batch that did not return every id it asked for — makes the run's
coverage partial, and no `missing` is asserted. Partial anywhere, on either side: the
run's completeness is one fact, not one per source, so a receipt never counts a finding
in a class the same receipt says it could not reach.

A page that **fails** is not a partial leg: it fails the whole run. The read's own
`read_result` is chained with its error class first, then the run stops and its row
settles `failed`. Nothing is released from a run that could not read what it set out to
read, so no verdict is minted off half a window — and the chain is left holding a valid
incomplete tail that a re-run of the same window converges onto.

Two classes are **declared not detectable** rather than denied: `backdated` needs a
persisted belief with per-entry first-observation evidence, and `edited_after_rekey`
needs persisted match state. Neither store exists on this shape, so their counters are
absent from the verdict and named in `not_detectable`. A present zero would attest "no
backdated entries" for a window nobody looked at.

No cardinality census runs here, so every correlation key reads **un-attested**:
`derivation: key` does not carry proven uniqueness.

A re-run of the same window shares its receipt's dedup key but chains a **new** record
whenever coverage or observed material moved, and nothing marks the earlier ones
superseded. Every surface renders the latest record and sets `earlier_records_exist`
when others are on the chain.

## A killed run, and what a resume may claim

A run that dies mid-flight leaves its row `running`, and the next reconcile of the same
window adopts that row and its id: identical intents re-chain, the chain adopts them,
and nothing pairs twice.

A run that dies **after** its `matcher_run` record landed is a different case — that
record is what says a run is finished, and it pins the input snapshot and the cut the
verdict was judged against. A resume re-reads both sources, so if the books moved (or
the day did) the fresh reads no longer match what that record attests. The resume then
**refuses** rather than sign a verdict with an environment that did not produce it: the
row settles, and the next reconcile of the window proves the fresh reads under the next
generation, with a run id of its own. Run the same command again and it goes through.

## One process at a time

A company's chain is held by **one** process. While an MCP session is up, `reconcile`,
`links-confirm`, `links-revoke`, `export`, `summary` and `evidence` are refused for that
company at the CLI: close the MCP client first. Lock order is chain, then links.

## Custody — read this before sharing anything

> link records carry the correlation key material that formed them — for
> exact-dimension links that includes the entry's date, amount and account set.

That sentence rides every surface that moves chain content off the machine — `export`,
`evidence`, `summary`, and the MCP `links_list` row. It is not a warning about a bug:
recording the key values that **formed** a link is what lets someone else check the
link, and what makes a link reviewable by a person. The chain stays in your own custody;
the disclosure exists because a pack handed to a third party hands those values over
with it.

**Copying `chain-<id>.db` or `links-<id>.db` carries the same material with no
disclosure attached.** Treat those files as you would the underlying books.

### Two honest limits

1. **Permanence.** The chain has no redaction, deletion or tombstone path. A key value
   on it is on it. The only remedy is a documented per-company chain reset, which
   destroys that company's read evidence along with the link material.
2. **Tamper-evidence.** Neither store is tamper-evident against its own operator. The
   projection's append-only property is advisory (SQLite has no role system), and a
   wholesale rewrite-and-rehash passes every internal check. The real protection is a
   chain head recorded **separately** at handover.

## Backups

`links-<id>.db` is a rebuildable projection of the chain — except for three tables a
rebuild **preserves** and never replays, because nothing can re-derive them:

- `key_attestations` — a census verdict is not on the chain.
- `match_outcomes` — a `no_candidate` verdict mints no receipt at all, and a refusal's
  candidate set lives in no receipt body.
- `runs` — the durable run row a status read stands on.

So back up **both** files. Losing the chain loses the evidence; losing the projection
loses those three tables and nothing else.

## Progress

`--progress` writes JSON-lines (`json`) or human lines (`plain`) on **stderr**; stdout
stays the response envelope. The default is `plain` at a terminal and `off` otherwise.
Phase, source, collection, pages, records, requests, rate-window waits and link counts
ride the stream while the run works.

Five rules bind it: a wait on the source's rate budget is reported as a wait; no progress
line is evidence and none is chained; `off` changes output only, never pacing, ordering
or receipts; no vendor record content appears in a line; and a run whose completion was
never recorded renders as UNKNOWN with its last chained evidence — never as "no such
run", never as a clean verdict.

A local failure the run cannot chain — the run's own row refusing the settle write —
rides the stream as a `note` on the settle line, so `json` stays parseable JSON-lines and
`off` stays silent. Under `off` the fallback is the honest one: the row is left
unsettled, and a later status read reports it UNKNOWN.

## Reviewing links

A link the matcher minted is a claim, not an approval. Three verbs read and move the
review state, all against local state only:

- `terakota links-list --company mybooks` — the correlation links and verdicts recorded
  for the company: the two source-native ids, the correlation key name and value that
  joined them, how the link was derived, whether a review has confirmed or revoked it
  and on which review surface, and — for a refusal — the candidate ids that were
  refused. No vendor call, no receipt; stdout **is** the response envelope, and there is
  no output flag. Filters: `--run-id`, `--derivation key|config`, `--confirmation
  none|human|revoked`, `--outcome`, `--from` / `--to` (the same closed spelling
  `reconcile` uses, `--to` inclusive), and `--cursor`. A filter value this shape cannot
  reach is **refused with the reason**, never answered with an empty list. With
  `--cursor` present, every other parameter except `--intent` must be absent — the
  continuation carries the filters its first page ran under.
- `terakota links-confirm --company mybooks --link <relationship_id>` — records a human
  confirmation. It chains **one** `link_reclass` receipt and then moves the local
  projection, in that order. The receipt records the review **surface** (`human:cli`),
  never an actor, and stamps consequence scope `none`: a link confirmed here authorizes
  no irreversible act, and the first action path that needs one has to ask for a fresh,
  scoped confirmation.
- `terakota links-revoke --company mybooks --link <relationship_id>` — withdraws it.
  Append-only: the link and its whole transition history survive, and the projection
  reads `revoked`. A revoked link denies the verdicts that stood on it — the next
  reconcile of a window covering it reports the pair as `pending_confirmation` rather
  than matched.

Both writing verbs require an interactive controlling terminal, and there is no flag
that bypasses it. Run one without a tty and it refuses in full, limit included:

> links review runs at a terminal: stdin is not an interactive controlling terminal, and
> there is deliberately no flag that bypasses this. It is a bar, not a proof — a pty can
> be faked by anything with shell access — so what it buys is that an agent cannot
> confirm a link by accident on its way past.

## From an MCP session

Three composite tools ride the MCP server, and reconcile there is **start-and-poll
only** — a synchronous reconcile over the agent transport is barred, because MCP hosts
time out where one blocking `terakota reconcile` does not.

- `reconcile_start` — takes `effective_from` / `effective_to` on the same closed
  posting-date spelling, returns a run id immediately and keeps running. **One run per
  company at a time**; a second start while one is live is refused as `run_in_progress`.
- `reconcile_status` — reports one run by id: its window, its coverage, its counts by
  verdict class, which classes that run could not reach, and the lineage of the link set
  the verdict stands on. A run whose completion was never recorded is reported as
  `unknown` together with its last chained evidence.
- `links_list` — the same link rows `terakota links-list` prints, read from local state.

`reconcile_status` and `links_list` execute no vendor call and record nothing. There is
**no confirm tool on this transport**, and no flag anywhere that adds one. There is also
no CLI verb yet that reads a run's own record; `links-list --run-id <id>` is how you
reach a run's links from the terminal.

v1.7.0 moved the tool registry, and one thing on this transport needs action after
upgrading: **existing MCP list cursors are invalidated — re-issue the list call without a
cursor.** See [install.md](install.md#upgrading).
