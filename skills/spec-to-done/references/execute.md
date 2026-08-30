# Execute and replan

This is an internal composite stage. Read this entire reference through EOF before applying it; a bounded preview is not a complete read, so continue from the next unread line before acting. Return to the local composite root immediately after it and continue without a user-facing stage handoff. Use only ordinary artifact inspection—files, contents, timestamps, task IDs, and checkpoints—never an executable detector or an external skill.

# Execute Plan

Carry out a plan one task at a time, keeping an honest record of what actually became true.

You are the **orchestrator**. You dispatch each task, verify the result against observable state, write the track, and decide whether the plan still holds. Verification and the track are always yours, whoever performs the work.

This skill is domain-neutral: it executes code work, research, writing, operational, and physical-world plans alike.

## Working directory

```
spec-interview/<slug>/
  SPEC.md      the contract      (mandatory; read-only here)
  PLAN.md      the strategy      (read; replaced by references/plan.md)
  TRACK.md    the record        (you are the only writer)
  REPORT.md    written by references/report.md
```

`SPEC.md` is mandatory. If it is absent, malformed, or its readiness is not recorded in `state.md` as `Verdict: Ready`, execution does not begin: do not dispatch a task, do not verify one, and do not replan. Preserve every existing artifact untouched and return control to the composite root for Specify. A PLAN or a TRACK is evidence of what was attempted, never a substitute contract.

## Read-first completion barrier

Immediately after appending any task record, **stop task execution** and complete this sequence before any other task or terminal route:

1. Set the inline `Gate` from current evidence. Its whole value is exactly `plan holds` or `replan required`, with no appended explanation. `partial`, `blocked`, and `failed` always mean `replan required`; `done` and `no_op` may also require it.
2. Invoke `references/plan.md`. For `plan holds`, run maintenance and keep the version. For `replan required`, replan immediately, increment the version for material change, and append one later named `replan done (plan version N)` or `replan exhausted` checkpoint for **each** triggering task.
3. Remove every attempted ID from actionable PLAN. Give every remainder the next unused numeric `T<number>` ID—never a suffix such as `T5R`—while every unattempted surviving task keeps its existing ID and outcome. Repoint or remove dependencies on attempted tasks.
4. Re-read PLAN and TRACK. Require exactly one `Plan version:` in PLAN, matching the latest named checkpoint. If an attempted ID or dependency remains, a required named checkpoint is absent, or any effective gate is `replan required`, repair it now. Do not dispatch or report.
5. If the reconciled PLAN has a future task, select the first dependency-ready task in its written order and continue the loop; REPORT is forbidden. A terminal route requires zero actionable PLAN tasks plus the applicable terminal condition.

There is no task-record-to-REPORT path. `replan exhausted` permits no later task dispatch without a matching verified reopening. A blocked or failed record preserves its stable `Blocker:` ID, cause, and resolution condition. Execution never closes `state.md` or authors the final response; after reconciliation it returns through the existing report route.

## Execution modes

Pick per task, in this order:

**Delegated (default).** One subagent per task. Costs more total tokens — the subagent re-reads context you already hold — and buys two things worth more: your context stays clean, so a long run never hits lossy compaction; and the track becomes the only memory channel between tasks, which is what keeps it honest. A track written inline duplicates your context and quietly rots.

**Inline.** When the host provides no subagents, when the user asks for it, or when the whole plan is two trivial tasks. Everything else in this skill is unchanged — including verifying the postcondition as a separate act from doing the work.

**Human handoff.** When the task must be performed by a person or outside your reach — a physical action, an approval, an access grant. Present the task, its `Done when`, and what evidence you need in plain language; never hand a person the structured return contract below. Ask only: did it happen, what is now true, and anything unexpected. **You** turn that answer into the track entry, recording it as `attested`. Never mark such a task done on the assumption it happened.

---

## The loop

```
read SPEC + PLAN + TRACK
   |
   v
select next unattempted task
   |
   v
pre-dispatch check / dispatch / verify
   |
   v
append task record with inline Gate
   |
   v
complete the gate
   |
   +-- Gate: plan holds
   |      |
   |      v
   |   run plan.md checkpoint maintenance
   |   remove attempted ID from future PLAN
   |   keep PLAN version unchanged
   |
   +-- Gate: replan required
          |
          v
       invoke plan.md in replan mode immediately
       remove attempted ID from future PLAN
       create a new ID for any remainder
       append a named replan checkpoint
   |
   v
re-read TRACK + PLAN + observable state
   |
   v
reconciled? -- no --> repair; do not dispatch, report, or return to root
   |
  yes
   |
   +-- future task exists --> select next task
   |
   +-- terminal state -----> invoke references/report.md
```

A task cycle is not complete when its task record is appended. It is complete only after the gate has been resolved, PLAN is future-only again, every required named checkpoint has been appended, and TRACK and PLAN have been re-read.

No status may skip this sequence.

### 1. Select the next task

Before selecting any task or invoking the reporter, reconcile the most recently recorded task.

Do not select, dispatch, report, or return to the composite root while any of these is true:

- a task's effective gate is `replan required`;
- an attempted task ID still appears in PLAN;
- a required named gate checkpoint is missing;
- a PLAN dependency points to an attempted task;
- PLAN and TRACK disagree about the current version, lineage, blocker, or remaining future work.

Repair the applicable interruption window first.

The first task in `PLAN.md` with **no track entry at all**, and whose `Depends on` tasks are all `done` or `no_op`.

**Never redispatch a task that already has any track entry.** A `partial`, `blocked`, or `failed` task is unfinished work whose remainder is not yet planned; re-running it repeats side effects. The replanner returns that remainder as a new task with a new ID, so the track and the plan never disagree about what has been attempted.

If a dependency is `blocked` or `failed`, do not proceed to its dependents. Go to the replan gate.

### 2. Check before doing

Before dispatching, check whether `Done when` already holds.

This costs one cheap check and closes the gap where a previous run was interrupted after its side effects but before the track was written. It also catches work someone else already did.

Which status you record depends on **why** the condition holds:

- Nothing ever attempted this task — it was already true → `no_op`.
- A previous run may have acted and died before recording → `done`, with the observed state delta and a note that it was reconstructed after an interruption. Recording that as `no_op` would erase a real change from the record.

When you cannot tell the two apart, assume the second. Losing a state delta is worse than over-reporting one.

### 3. Dispatch

The performer starts cold, so the brief must stand alone. Send exactly this:

```markdown
## Task
<the full task block from PLAN.md — T-id, Task, Done when, Verify by, Covers>

## Goal context
<the one-sentence goal from PLAN.md>

## Constraints that apply
<constraints, business rules, and non-goals from the contract that bear on this task —
not the whole contract>

## Established facts
<the [verified] Discovered entries from TRACK.md that this task depends on —
identities, paths, versions, decisions already made — plus any fact from the task's
Reasoning the performer needs; for T1, the grounding observation. A fact still
[reported, unconfirmed] is not established: confirm it first, or pass it explicitly
labeled as an unconfirmed report>

## Return contract
Do the work, then verify your own postcondition and return ONLY this block:

status: done | partial | blocked | failed | no_op
state_delta:
- <what is observably different now, and its consequence>
evidence:
- <check performed> → <result>
blocked_because:
- <current observable condition preventing progress; required for blocked>
resolution_condition:
- <what must become observably true before continuation or reopening;
  required for blocked and failed>
failure:
- <observable discrepancy that prevented the postcondition; required for failed>
discovered:
- <fact that later tasks need; omit the section if none>
unresolved:
- <what remains undone; omit the section if none>
risk:
- <what could still go wrong as a result of this work; omit if none>
user_action:
- <what the user must do; omit if none>
deviation: <how the executed work differs from the task as written, and why; omit if none>

Rules:
- Report state, not activity. "Requests over 30s now fail" — not "I edited the client".
- One bullet per material fact, and omit any section that has none.
- Evidence is `<check> → <result>`, never a pasted log or full stdout.
- Do not repeat the task, the contract, an acceptance criterion, or your reasoning.
- State any destructive or non-repeatable effect explicitly in `state_delta`.
- Do not claim a check you did not run.
- A `discovered` item names what you observed and where, so it can be confirmed.
- Use `no_op` if the postcondition was already satisfied before you started.
- If the task is impossible as written, return `blocked` or `failed` with the reason.
  Do not improvise a different task.
- `blocked_because` and `resolution_condition` are mandatory for `blocked`;
  `failure` and `resolution_condition` are mandatory for `failed`.
- `user_action` appears only when the user must act. It never replaces the blocker
  cause or the resolution condition.
- Never invent the stable `Blocker:` ID. The orchestrator derives it from the work
  slug and `Root`.
- Stop and return `blocked` before any destructive or irreversible action the user
  has not explicitly asked for.
- Do not write to TRACK.md, PLAN.md, or the contract.
```

Give the performer the smallest slice of the contract that matters. Dumping the whole SPEC into every brief defeats the purpose of dispatching.

The brief deliberately omits the task's `Reasoning` — that is the planner's justification, not an instruction. If the reasoning contains a fact the performer needs, that fact belongs in `Established facts`.

### 4. Verify — do not take the performer's word

The performer is not the judge of its own success. Before writing the track, check the postcondition yourself against observable state:

- Run the `Verify by` check, or confirm the reported evidence is real and current.
- Confirm the `Done when` condition actually holds now.
- If `done` was claimed but the postcondition does not hold, record `failed` with the discrepancy. Do not soften it.

Extend the same skepticism to `discovered:` items — they become later tasks' input and the replanner's premises. Confirm each one against observable state when a cheap check exists and record it `[verified]`; when you cannot, record it `[reported, unconfirmed]`. An unconfirmed discovery is a hypothesis, not a fact.

Label the verification, because the reporter depends on the distinction:

| Label | Meaning |
|---|---|
| `verified` | You ran or confirmed a check that observes the postcondition |
| `attested` | A person confirmed it; valid evidence, not machine-checked |
| `unverified` | No check was possible, or confirmation is still pending |

`attested` is a satisfied criterion. `unverified` is not.

**Never record `done` with `unverified`.** If the postcondition cannot be observed at all, the status is `partial` and the unobservable condition goes in `Unresolved`. Otherwise the executor would call a task finished that the reporter must then downgrade, and the run would end claiming a completion the evidence does not support.

`Done when` is conjunctive. Record `done` only when every required clause of the postcondition is currently observable and supported by `verified` or `attested` evidence.

- If a check disproves any required clause, record `failed`.
- If a required clause cannot be observed, record `partial`, `Verification: unverified`, and name the exact missing observation under `Unresolved`.
- Do not infer reload, restart, persistence, propagation, external side effects, or user-visible behavior merely from a file edit, implementation state, or an unrelated passing check.
- A task supports a covered Must requirement or acceptance criterion only to the extent that its evidence observes the required behavior.

### 5. Write the track entry

**You are the only writer.** Performers return structured blocks; you validate and append. One writer means one format, no concurrent writes, and a natural point to decide on replanning.

Append to `spec-interview/<slug>/TRACK.md`. Historical TRACK records may use equivalent headings and wording when their meaning is unambiguous; preserve them, never rewrite them. Every new record uses the canonical field names below, and does not rename, merge, or omit a field its status requires.

```markdown
# TRACK: <slug>

## T1 — Payment client timeout  [done]
Plan version: 1
Covers: FR-004, AC-002
Root: T1
State delta:
- Outbound payment requests now fail after 30 seconds instead of hanging indefinitely.
Evidence:
- integration test PaymentTimeoutTests → passed
Verification: verified
Gate: plan holds

## T2 — Product endpoint cache  [partial]
Plan version: 1
Covers: FR-007, AC-005
Root: T2
State delta:
- The cache abstraction exists and the product endpoint reads through it.
Evidence:
- unit tests → passed (14)
- production Redis behavior → not checked, no access to that environment
Verification: unverified
Discovered:
- [verified] The product endpoint is also called by the reporting job, which expects fresh data.
Unresolved:
- Cache invalidation for the reporting path.
Risk:
- The reporting job may read stale data until invalidation is added.
Deviation: TTL set to 60s rather than the planned 300s, because the reporting job
tolerates at most one minute of staleness.
Gate: replan required

### Replan checkpoint — T2
Previous plan version: 1
New plan version: 2
Gate: replan done (plan version 2)

## T4 — Publish production release  [blocked]
Plan version: 2
Covers: FR-009, AC-014
Root: T4
Blocker: BLK-release-T4
Blocked because:
- Production deployment credentials are unavailable to this run.
Resolution condition:
- A production credential check succeeds, or matching access is explicitly attested.
Evidence:
- credential inspection → no production credential available
Verification: verified
User action:
- Grant production deployment access.
Gate: replan required
```

`Blocker` identifies the stable lineage obstacle. `Blocked because` records its current observable cause. `Resolution condition` defines the evidence required to continue or reopen. `User action` exists only when the user must act. None of these four substitutes for another.

TRACK rules:

- **Canonical fields, mandatory by status.** Every task record carries task identity, status, plan version, coverage, lineage, evidence, verification, and its inline `Gate:`, plus every field the status table below requires — blocker, blocked because, resolution condition, failure, state delta, discovery, unresolved item, risk, user action, or deviation. Write those field names as they appear here.

- **The initial gate is inline: every task record ends with its own `Gate:` line,** holding `plan holds` or `replan required`. Writing it inside the record is what makes the association positional, so it can never be read as belonging to another task.

- **Every later gate transition is a separate checkpoint that names the task,** carrying the plan version and the new value — `replan done (plan version N)` or `replan exhausted`. The current gate for a task is the last gate event associated with it.

- **A gate that names no task is a format error.** One gate covering several tasks at once, or a trailing global gate at the end of the file, destroys the task-to-gate association the report is built from. It is not untidy style; it is unusable evidence.

- **Append every task entry and every gate checkpoint.** `TRACK.md` is append-only: never edit a prior task or checkpoint, including to revise a gate. A gate changes by appending a new checkpoint, never by rewriting the old value. A later fact is a correction checkpoint naming the affected task; history is never rewritten.
- **Write it after every task, never reconstruct it at the end.** A track rebuilt from memory at the end of a long run is exactly the semantic loss this workflow exists to prevent.
- **Copy `Root`, and any `Continues` or `Reopens`, verbatim from the plan.** They are the lineage record; without them TRACK cannot show which attempt followed which, and the blocker loses its stable identity.
- **Copy `Covers` verbatim from the plan.** It is the only path from task evidence back to acceptance criteria once the task leaves the plan. Every task has one; a task without it is a plan defect, not a run to continue.
- **Every `Discovered` item carries its provenance label** — `[verified]` when you confirmed it against observable state, `[reported, unconfirmed]` when it is only the performer's claim. Briefs, replans, and the gate treat the two differently, so an unlabeled discovery is a format error.
- **State over activity.** "Authentication now rejects expired tokens" — not "edited AuthService". Filenames are optional traceability, appended after the consequence.
- **Record deviations honestly.** A silent deviation becomes a hidden contract change.
- **Destructive or irreversible actions and changes to user data are always state deltas.** Never leave one implicit.
#### Retention lanes

Every fact you are about to write belongs to exactly one of these three lanes. The record is written short at the point of writing, never summarized later — nothing already appended is ever compressed again, so a fact dropped here is lost for good.

**Protected — never omit.** Task ID, status, plan version, `Covers`, `Root`, `Continues`, `Reopens`, blocker ID, blocker cause, resolution condition, pending user action, any non-repeatable effect or repeat prohibition the SPEC or the task states, `Verification`, provenance labels, `Gate`, and every exact path, version, token, identifier, or number whose value changes what a later task does. Identifiers and labels are copied literally. Other protected facts may be written concisely, but their operational meaning stays complete: a blocker cause and a resolution condition are operational state, not reasoning, and are never removed for concision. When it is unclear whether a value must stay literal, it must.

**Compact — reduce to material consequence.** State delta, Evidence, Discovered, Unresolved, Risk, and Deviation carry only facts that remain operationally relevant. One bullet per material fact, and never the same proposition in two fields: the delta owns the consequence, "expired tokens are now rejected", and the evidence owns the check that proves it, "AuthTests → 18 passed". Restating one sentence in both is duplication; the delta and the evidence pointing at the same result from their own angle is not. There is no bullet limit — most fields need one to three, but a protected fact is never merged or dropped to reach that size.

**Drop — never persist.** Reasoning, execution narrative, raw logs, full stdout, restated task text, restated contract text, and recovered transient errors with no residual consequence. Never paste the performer return into TRACK. An error that changed the final state is a state delta, not a dropped one.

#### Mandatory content by status

| Status | Mandatory TRACK content |
|---|---|
| `done` | State delta, Evidence, Verification, Gate |
| `no_op` | Evidence showing the condition already held, Verification, Gate |
| `partial` | Evidence, Verification, Unresolved, Gate; State delta when something changed |
| `blocked` | Blocker, Blocked because, Resolution condition, Evidence, Verification, Gate; User action only when applicable |
| `failed` | Blocker, Failure, Resolution condition, Evidence, Verification, Gate; State delta when something changed |

A section with nothing to say is left out only when this table does not require it.

Before appending, confirm:

- if status is `blocked`, do not append unless `Blocker`, `Blocked because`, `Resolution condition`, `Evidence`, `Verification`, and `Gate: replan required` are present;
- if status is `failed`, do not append unless `Blocker`, `Failure`, `Resolution condition`, `Evidence`, `Verification`, and `Gate: replan required` are present;
- if status is `partial`, do not append unless `Unresolved` is present;
- every identifier, lineage field, provenance label, non-repeatable effect, repeat prohibition, and the effective `Gate` are present and literal;
- every `Evidence` line reads check → result, in the evidence class it came with;
- no reasoning, raw log, restated task or contract text, or repeated prose remains;
- if any required item is absent, rewrite the record before appending.

This check is transient. Do not write it, its result, or a verdict to TRACK.

### 6. Status transitions

**This table is normative.** Where the diagram, the prose, or the anti-patterns seem to say otherwise, the table wins. Every status reaches the gate, and no status leaves the loop undefined. The resulting `Gate`, not the status alone, selects checkpoint maintenance or material replanning.

Record contents are set by the mandatory-content table in step 5; this table sets only what happens next.

| Status | Then |
|---|---|
| `done` | replan gate → next task |
| `no_op` | replan gate → next task |
| `partial` | replan gate **must** run — the remainder needs a new task |
| `blocked` | replan gate **must** run; if it cannot route around it → report |
| `failed` | replan gate **must** run |

### 7. Replan gate

Run this after **every** task, without exception. It is a checkpoint, not a rewrite: most gates should pass without a replan.

Ask: *is the remaining plan still true, given what is now known?*

Invoke `references/plan.md` in replan mode when any of these holds:

```
- the task returned partial, blocked, or failed
- a postcondition failed and a retry will not fix it
- an expected file, resource, person, capability, or state does not exist
- a discovered fact, confirmed [verified], invalidates a later task
- a task turned out to be impossible as written
- a materially shorter valid path became available
- a new constraint surfaced during execution
- an acceptance criterion requires work no task covers
- a deviation changed what later tasks can assume
```

Otherwise state that the plan still holds and continue.

**Unconfirmed discoveries never redraw the strategy.** If a `[reported, unconfirmed]` fact would invalidate later tasks, confirm it first; if confirming it takes real work, the only replan it may drive is one that adds a validation task. The strategy rewrite waits for that task's verified result.

**Mid-task trigger.** Do not wait for the task to end when the divergence is already fatal. If a performer reports `blocked` or `failed` with a reason that invalidates the plan, replan immediately rather than dispatching the next task into a plan you know is wrong.

### Complete the gate before continuing

`Gate: replan required` is an intermediate state, never a terminal state.

While it is the effective gate for a task, it is forbidden to:

- select or dispatch another task;
- invoke `references/report.md`;
- classify the run as terminal;
- return control to the composite root.

After every task record is appended:

1. Evaluate the replan gate.
2. Invoke `references/plan.md`; the executor never edits PLAN directly.
3. If the gate is `plan holds`:
   - run checkpoint maintenance;
   - remove the attempted task from actionable PLAN;
   - keep the current PLAN version;
   - append no replan checkpoint.
4. If the gate is `replan required`:
   - invoke `references/plan.md` in replan mode immediately;
   - remove the attempted task from actionable PLAN;
   - give any remainder a new ID;
   - preserve surviving IDs and repoint dependencies as already specified;
   - append exactly one named checkpoint for the triggering task:
     - `replan done (plan version N)` when a valid future-only PLAN is ready;
     - `replan exhausted` when no valid continuation remains under the existing contract and attempt rules.
5. Re-read PLAN and TRACK before selecting work or reporting.

A `partial`, `blocked`, or `failed` task with `Gate: plan holds` is a format error. A `done` or `no_op` task may still require material replanning when a verified discovery, deviation, constraint, or coverage gap invalidates the remaining strategy.

Before continuing or reporting, confirm:

- no task ID already present in TRACK remains in actionable PLAN;
- no PLAN dependency points to an attempted task;
- every inline `replan required` has a later named transition associated with the same task;
- no task has `replan required` as its effective gate;
- PLAN version and the latest named checkpoint agree;
- surviving tasks remain ordered and their dependencies are acyclic;
- Root, Continues, Reopens, blocker identity, episode, and attempt limits remain unchanged;
- SPEC, PLAN, TRACK, and observable state do not contradict one another.

This check is transient. Do not append it or its verdict to TRACK.

If any item fails, repair the applicable interruption window before selecting, dispatching, reporting, or returning to the composite root.

### 8. Exit

A terminal route is legal only after the post-task reconciliation above passes.

`replan required` cannot be reported as terminal. A blocked or failed lineage that has no valid continuation reaches REPORT only after its named `replan exhausted` checkpoint exists.

Stop the loop and invoke `references/report.md` when:

- every task is `done` or `no_op`; or
- the plan has zero tasks because the goal was already satisfied; or
- a task is `blocked` and replanning cannot route around it; or
- the plan cannot be repaired without changing the contract; or
- the user asks to stop.

Never stop silently. Every exit goes through the reporter.

Execution does not construct or paraphrase the terminal response. Invoke `references/report.md` through the existing composite route. After REPORT is persisted and the composite root closes `state.md`, emit exactly the persisted report body byte-for-byte. Do not reconstruct Markdown, add narration, or replace the persisted body.

---

## Escalation

Stop and ask the user — do not decide alone — when:

- an acceptance criterion has become unreachable;
- an action would be destructive or irreversible and the user has not explicitly asked for it;
- the work has drifted far enough from the contract that finishing it would satisfy a different goal;
- a blocker needs access, credentials, or a decision only the user has.

Report the state through `references/report.md` rather than improvising a new objective.

---

## Resuming

`TRACK.md` is the resume point. On restart, read the contract, `PLAN.md`, and `TRACK.md`. First reconcile each interruption window without changing any existing entry: execution-to-TRACK uses the step 2 pre-dispatch state check before recording; TRACK-to-gate appends a missing checkpoint for the recorded task; gate-to-PLAN compares the last checkpoint's plan version and gate before any continuation is created. If `replan required` has a higher `PLAN.md` version, append `replan done (plan version N)`; at the same version invoke the replanner. Never dispatch a recorded task ID.

If the last checkpoint says `replan exhausted`, it is terminal until resolution of its `Blocker: BLK-<slug>-<root-task-id>` is verified by ordinary inspection or explicitly attested by the user. The evidence must match that stable blocker identity. Append the resolution evidence and a reopening checkpoint, invoke the replanner, and start a same-SPEC new `Reopens:` episode under the same `Root:`; preserve all earlier entries and retain the exhausted lineage as historical evidence.

Do not re-run completed tasks. Re-verify a completed task only when a later discovery may have invalidated its postcondition.

The same completion barrier applies after a restart.

After repairing execution-to-TRACK, TRACK-to-gate, or gate-to-PLAN, re-read TRACK and PLAN and run the reconciliation check above. Do not select work or report merely because the interrupted task already has a TRACK entry.

A higher PLAN version closes `replan required` only after the corresponding named `replan done` checkpoint is appended. The same PLAN version with `replan required` still invokes the replanner. `replan exhausted` remains terminal until its matching resolution and reopening rules are satisfied.

---

## Anti-patterns

**Trusting the return.** Accepting `status: done` without checking the postcondition. The performer is optimistic about its own work; that is the whole reason for the verification step.

**Redispatching unfinished tasks.** Re-running a `partial` task from the top repeats its side effects. The remainder is new work and needs a new task.

**Batch track writing.** Running five tasks and then writing five entries from memory. The details that mattered are already gone.

**Activity tracks.** Entries that list what was touched instead of what became true. The report cannot be built from those.

**Skipping the replan gate when things are going well.** The plan is most often wrong exactly when execution feels smooth — because nothing has forced you to look at it.

**Fixing the plan inline.** When the plan is wrong, invoke the replanner. Silently doing something other than what the plan says produces a run nobody can audit.

**Reinventing the todo list.** `PLAN.md` is the durable artifact. Any ephemeral task-tracking UI is a view of it, never a second source of truth.


## Full protocol

Every task record carries `Root: <origin task ID>`; a task that starts a lineage names itself. Every replan continuation adds `Continues: <immediate attempted predecessor>`, and a reopening adds `Reopens: <last exhausted attempt>` in its place. All three are copied verbatim from the plan into the track entry.

`Continues:` reconstructs the real sequence of attempts; `Root:` is what stays stable, so it — never the immediate predecessor — supplies the `<root-task-id>` in the blocker identity.

A lineage permits one root attempt and at most two continuation attempts per episode, for a maximum of three attempts. T1 is the root, T2 and T3 are continuations, and T4 is forbidden in the same episode. Its checkpoints expose `root_attempts: 1`, `continuation_attempts: 0–2`, `continuation_limit: 2`, `total_lineage_attempts: 1–3`, and `total_lineage_limit: 3`. A blocked or failed lineage records and reuses `Blocker: BLK-<slug>-<root-task-id>`. An exhausted episode may reopen only after ordinary inspection verifies, or the user explicitly attests, resolution of that same blocker. Append the evidence and a `replan reopened (plan version N)` checkpoint, preserve prior history, and start a new `Reopens:` episode rather than extending the exhausted lineage. Before any new work reconcile each window: execution-to-TRACK by inspecting side effects before recording, TRACK-to-gate by appending a missing checkpoint without redispatch, and gate-to-PLAN by comparing plan versions and gates before creating a continuation. Never repeat a recorded task ID or a completed side effect.
