# 0046 · An app is a declared agent

- **Status**: cut 1 · this record. No code yet
- **Written**: 2026-09-19 (conventions in the [README](README.md))
- **Related**: [0042](0042-the-chat-loop-is-a-runner-with-hooks.md) is the runner an app
  instantiates; [0034](0034-an-action-is-a-declared-call.md) is the only way an app reaches the
  world, and its parameter design is reused here; [0014](0014-identity-from-the-person-scope-from-the-token.md)
  is the intersection this record extends by one term; [0015](0015-recording-a-sentence-is-not-asserting-a-fact.md)
  is why an app's writes wait; [0019](0019-the-second-clock-can-be-rewound.md) is what makes a
  rerun mean anything; [0020](0020-an-auditor-reads-it-without-us.md) is why every run is a row.
  The grant layer follows `data_source_grants` (#142) and 0034's `action_grants`.

> A person works out, over a week, which tools and which definitions get a trustworthy answer
> about contracts expiring this quarter: ask the graph first, read the definition of *term end*
> before the numbers, never trust a date the document did not state. That week is worth something
> to the eleven other people who will ask the same question. Today it can be handed over only as a
> paragraph of advice in a chat message. The base has a runner, a tool surface, a queue and a
> ledger; it has no way to name a configuration of them, give it to a team, and run it again.

## What the ground already gives, and what it withholds

Five parts are reusable as they stand:

- **A runner with hooks.** The loop is rig's, policy is one `AgentHook`, and `chat::tools_schema`
  is the single place a tool list is built (0042).
- **A tool surface with its boundary in one struct.** `dispatch` in `api/tools.rs` routes ten read
  tools and `remember`; what a turn may touch is `ToolCtx` — the base, the mounted sources, the
  actor, the token, `can_write`. Nothing reads around it.
- **The intersection rule.** Effective permission is already the person's role in the base
  intersected with the token's scope (0014, `tokens.rs`).
- **A queue and a schedule vocabulary.** `jobs` claims with `SKIP LOCKED`, dedupes on
  `(kind, payload)`, honors `run_at`, and scales with `worker_concurrency`; sources already parse
  a five-field cron.
- **Two ledgers.** `audit_events` for who did what, and 0034's `action_runs` for what left the
  deployment.

What it withholds: no table holds a configuration of those parts, `tools_schema` takes a boolean
and a list of sources rather than a policy, and nothing can run a turn with no browser attached.

## Decisions

**1. An app is data.**

An app is a name, a description, instructions in prose, a list of tools it may call, the
definitions it pins, how it resolves its clock, an output shape, an optional action and a list of
typed parameters. The registry page renders it from the same rows the runner reads.

This record owes 0034 an answer, because 0034 refused expressions and conditionals on the grounds
that what a person authors should be structured enough that the page shows what runs — and an app
holds a prompt, which is prose. The line: **everything that decides what an app may touch is
structured** (the tools, the base, the definitions, the action, the clock), and only **what it is
asked to do** is prose. A prompt is an instruction to a model, never a grant. No sentence in it
widens reach, because reach is computed from rows before the first token is generated.

**2. An app runs as the person who runs it.**

The tools a run may call are the app's list intersected with what the caller could have called by
hand: their role in that base, and over MCP the token's scope as well. The author's own role never
enters it.

The alternative — an app carrying its author's reach, the service account every platform grows —
was refused. It makes `require_kb` decorative: a Viewer would read a restricted base through an
app an Editor published, and the matrix would still show them as a Viewer. An app center is the
newest surface in the product and would become the widest hole in it on the first day.

**3. An app narrows and never widens.**

The tool list subtracts. `remember` stays behind `can_write`, `query_data` stays behind what the
base has mounted, and an app cannot be opened in a base it was not granted. An empty tool list
means *everything the caller already had*, so an app is useful before its author has thought about
tools, and a tool added to the product later does not silently appear inside old apps that named
their tools.

**4. A run declares its clock.**

An app resolves `as_of` one of two ways: the moment the run starts, or a parameter the person
supplies. The resolved instant is passed to every graph read in that run (0019 already accepts it
on all of them) and written on the run row.

Without this an app answers "now" and its answer cannot be got again, which gives away the one
promise this product is built to make. With it, the log row is enough to reproduce a number a
person acted on three months ago, and an app can be pointed at a quarter end on purpose.

**5. The world is reached through a declared action.**

An app holds no URL and gets no fetch tool. One that must open a ticket or post a message binds an
action granted to that base, and the call is an `action_runs` row like any other (0034). Otherwise
the pinned addresses, the re-checked redirects, the sealed credentials and the outbound ledger are
all bypassed by whichever surface is newest.

**6. A trigger is a person or a schedule.**

A scheduled run has no caller, so it runs as **the person who armed it**, named on the row. Arming
needs Editor in that base; when that person loses the role the schedule disarms and raises an
alert. A schedule that outlives its owner's access is decision 2's hole reached by a slower route.
Events (a derived row first appearing, a queue crossing a threshold) are the third trigger and are
not in this record, because 0034 already describes a rule firing an action and the two should be
designed in one pass.

**7. Every run is a row.**

`app_runs` records the app by id and by name, the base, the trigger, who, the arguments, the
resolved `as_of`, the steps, the answer, the citations, the action run if there was one, the token
counts, the duration and the error. The base's log and the deployment's log read the same table.
The token counts are there because a scheduled app is the first thing in this product that spends
money while nobody watches.

**8. A write still waits for a nod.**

An app that calls `remember` writes a pending statement and a person confirms it (0015). An app is
a faster way to ask, never a way to assert.

**9. Runs go through the existing queue.**

A scheduled run is a `jobs` row, deduped on `(app_id, scheduled_for)`, so a worker restart or a
schedule read twice sends one run. An interactive run streams on the request like chat does. There
is no second executor and no second concurrency knob.

**10. An app is authored in a base and granted beyond it.**

A base's Editor writes an app there, because writing one means running it against real data until
the answer is right. Granting it to other bases is a deployment admin's act, since the instructions
encode assumptions about the base they were written against. A Viewer runs a granted app and sees
the log; nobody sees an app in a base it was not granted to.

## Dead ends

**Code in this record.** An app someone writes code for is a second kind of thing and gets its own
record (0047), because its criteria are different: a capability is an imported function rather than
a tool name checked at run time, determinism is a requirement rather than a nicety, and the author
is a coding agent rather than a person. What is settled here and carries over: the identity rule,
the egress rule, the declared clock, the run row and the queue.

**A container as the first backend for that tier.** Withdrawn the day it was written. It was
argued from WeKnora, whose skills are written by people, installed from a catalog and assume a
shell — so they need an operating system, and WeKnora pays for it (the host-process backend
removed outright, the Docker backend made opt-in because a mounted `docker.sock` is host root,
every exec moved off root to uid 1000, symlink escapes out of the workspace closed). A component
this product's own coding agent writes against a typed interface has no such requirement, and the
things a container costs — a daemon beside the binary, a hundred milliseconds of start-up per run,
and an ambient clock, network and filesystem that make a re-parse non-reproducible — are all
things this product would rather not pay. A container backend stays possible behind the same trait,
off by default, for the customer who arrives holding a Python script.

**An app as a saved conversation.** Rerunning a saved chat would replay a resolved question — the
entities it had already identified, the month it was asked in. An app carries its instructions and
its clock, and meets the data fresh.

**A per-app credential.** An app that holds a key of its own is a service account wearing a
different word, and decision 2 refused it once already.

## Schema

```sql
apps          id, workspace_id, name (unique per workspace), description,
              instructions TEXT, tools TEXT[] (empty = whatever the caller may call),
              definitions UUID[] (concept_mappings pinned into the prompt),
              clock ('run_time' | 'parameter'), output ('text' | 'table'),
              action_id (nullable, must be granted to the same base at run time),
              enabled, home_kb_id, created_by, created_at, updated_at
app_params    id, app_id (cascade), seq, name (^[a-z][a-z0-9_]{0,39}$, unique per app),
              kind (number | string | boolean | date), required, description,
              min_value, max_value, options TEXT[]
app_grants    app_id (cascade), kb_id (cascade), granted_at, granted_by
app_schedules id, app_id (cascade), kb_id, cron TEXT, timezone, args JSONB,
              armed_by, enabled, last_run_at, next_run_at
app_runs      id, app_id (set null on delete), app_name, kb_id (set null), trigger
              ('person' | 'schedule'), actor_id, via_token, args JSONB,
              as_of TIMESTAMPTZ NOT NULL, steps JSONB, answer TEXT, citations JSONB,
              action_run_id, status, error, prompt_tokens, completion_tokens,
              duration_ms, started_at, finished_at
```

Parameters are a table with the same columns and the same checks as `action_params`, for 0034's
reasons: the database keeps the bounds rather than the store remembering to, and a later binding
can point at a parameter by id and survive a rename. `tools` is an array rather than a table
because an entry is a tool name the code already spells, and a name that no longer exists should
read as absent rather than as a dangling row.

## API

```
GET    /kbs/{id}/apps                       granted here, enabled, without instructions for a Viewer
POST   /kbs/{id}/apps                       Editor; the app is granted to this base on creation
PATCH  /kbs/{id}/apps/{app_id}              Editor, in the home base only
POST   /kbs/{id}/apps/{app_id}/run          SSE, the same frames as chat plus a run id
GET    /kbs/{id}/apps/{app_id}/runs         this base's log
POST   /kbs/{id}/apps/{app_id}/schedule     Editor; armed as the caller
GET    /admin/apps                          the deployment's registry
POST   /admin/apps/{app_id}/grants          deployment admin
GET    /admin/app-runs                      across every base
```

An app is not a new protocol: over MCP a granted app appears as a tool whose parameters are its
own, which is the whole of what an external agent needs from this record.

## Open questions

- **Output shapes beyond text and a table.** A chart is the obvious third, and it belongs to
  whatever the interface does for charts rather than to this record.
- **Cost.** A scheduled app spends tokens with nobody watching. The run row counts them; a budget
  that stops an app has no surface today except governance's daily cap, and pricing a team's usage
  is not designed.
- **An app calling an app.** Cheap to allow and hard to bound; left out until something asks.
- **Events as a trigger**, together with 0034's rule-fires-an-action seam.
- **Where a team meets an app.** This record gives an app a page inside the product. WeKnora's
  equivalent surfaces are an embed widget with a domain allowlist and a rate limit, the IM channels,
  and a scoped API key — the app center is where an app is administered and those are where it is
  used. Which of them this product wants is not decided here; MCP is the one it already has.
