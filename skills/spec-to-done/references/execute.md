# Execute and replan

This is an internal composite stage. Read this entire reference through EOF before applying it; a bounded preview is not a complete read, so continue from the next unread line before acting. Return to the local composite root immediately after it and continue without a user-facing stage handoff. Use only ordinary artifact inspection—files, contents, timestamps, task IDs, and checkpoints—never an executable detector or an external skill.

# Execute Plan

Carry out a plan one task at a time, keeping an honest record of what actually became true.

You are the **orchestrator**. You dispatch each task, verify the result against observable state, write the track, and decide whether the plan still holds. Verification and the track are always yours, whoever performs the work.

This skill is domain- and provider-neutral: it executes code work, research, writing, operational, and physical-world plans alike without depending on a vendor-specific runtime.

## Working directory

```
spec-interview/<slug>/
  SPEC.md      the contract      (mandatory; read-only here)
  PLAN.md      the strategy      (read; replaced by references/plan.md)
  TRACK.md    the record        (you are the only writer)
  REPORT.md    written by references/report.md
```

`SPEC.md` is mandatory. If it is absent, malformed, or its readiness is not recorded in `state.md` as `Verdict: Ready`, execution does not begin: do not dispatch a task, do not verify one, and do not replan. Preserve every existing artifact untouched and return control to the composite root for Specify. A PLAN or a TRACK is evidence of what was attempted, never a substitute contract.

Execute coordinates the post-task transition but never owns PLAN content. It supplies the complete inputs to `references/plan.md`, validates the returned PLAN, and writes only TRACK task records and checkpoints. If the planner result is invalid, invoke planning again; never reconstruct, patch, or replace PLAN inside Execute.

## Read-first completion barrier

Immediately after appending any task record, Execute enters the post-task critical
section defined in **Record and close one task** below. There is no legal exit to
a new task, the composite root, or REPORT until that same triggering task has a
future-only PLAN, an effectively closed gate, every required named checkpoint,
and a successful `POST-TASK CLOSED(trigger)` check.

This is a control-flow critical section, not an atomic filesystem transaction.
A physical interruption may occur between writes; on resume, repair that window
before any other work. Plan alone writes PLAN. Execute alone writes TRACK task
records and checkpoints and never repairs PLAN itself.

There is no task-record-to-REPORT path. `replan exhausted` closes only the triggering `Root:` and episode. Until a matching verified or attested resolution and reopening checkpoint exist, no same-Root continuation or task that still depends on the exhausted lineage may be dispatched; an evidence-independent task under another Root may continue. Exhaustion is terminal for the run only when every named checkpoint is valid, no matching resolution is pending, and PLAN contains no eligible future task. Execution never closes `state.md` or authors the final response; after reconciliation it returns through the existing composite route. Until these conditions hold, REPORT is forbidden.

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
   |   plan.md writes checkpoint-maintained PLAN
   |   PLAN version stays unchanged
   |   no checkpoint is appended
   |
   +-- Gate: replan required
          |
          v
       plan.md writes material PLAN or returns exhaustion
       Execute validates the result
       Execute appends the required named checkpoint
   |
   v
re-read TRACK + PLAN + observable state
   |
   v
reconciled? -- no --> repair; do not dispatch, report, or return to root
   |
  yes
   |
   v
return to composite root
   |
   +-- future task exists --> root selects Execute
   |
   +-- terminal state -----> root selects Report
```

A task cycle is not complete when its task record is appended. It is complete only after the gate has been resolved, PLAN is future-only again, every required named checkpoint has been appended, and TRACK and PLAN have been re-read.

No status may skip this sequence. One Execute invocation handles at most one task and its complete reconciliation; never preselect or batch multiple tasks.

### 1. Select the next task

When the root selects Execute, reconcile the most recently recorded task before selecting one new task.

Do not select, dispatch, report, or return to the composite root while any of these is true:

- a task's effective gate is `replan required`;
- its required named checkpoint is missing or names a different task;
- an ID already present in TRACK still appears in PLAN;
- a PLAN dependency points to an ID already present in TRACK;
- PLAN declares `Status: no-op` after TRACK contains any task record;
- PLAN does not contain exactly one `Plan version:` field;
- PLAN and TRACK disagree about version, survivor identity, order, lineage, blocker, or remaining future work;
- a same-Root continuation or task still dependent on an exhausted lineage was dispatched before matching verified or attested reopening.

Repair the applicable interruption window first.

Any replacement or new outcome uses the next unused numeric task ID, never a suffix such as `T5R`.

Scan PLAN task blocks from top to bottom on every selection. For each task in written order:

1. If its ID already has any TRACK task record, PLAN is invalid; stop selection and reconcile instead of skipping it.
2. Evaluate every `Depends on` ID. The task is eligible only when each dependency is satisfied by `done` or `no_op` evidence, or is absent because checkpoint maintenance already removed that satisfied dependency.
3. Select the first eligible task and stop the scan. Never preselect a later task, choose a more convenient ready task, or reorder PLAN during selection.
4. If a task is not eligible, continue downward only to find the first later task whose prerequisites are independently satisfied. If none exists, reconcile or take the valid terminal route; do not dispatch a blocked dependent.

If TRACK contains any task record, reject any PLAN that declares `Status: no-op`. That field is legal only for the initial zero-task, already-satisfied plan before execution; it can never summarize, erase, or terminate recorded work.

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

### 5. Record and close one task

**You are the only writer.** Performers return structured blocks; you validate and append. One writer means one format, no concurrent writes, and a natural point to decide on replanning.

Complete one triggering task in this exact order:

```text
assemble task record
-> validate mandatory fields
-> append task record
-> enter post-task critical section
-> invoke Plan
-> validate PLAN and its internal receipt
-> append the required checkpoint, if any
-> re-read PLAN + TRACK
-> prove POST-TASK CLOSED(trigger)
-> return to the composite root
```

Once the task record is appended, no status, planner result, empty PLAN, user
stop, or apparent terminal condition may skip the remaining sequence.

Append to `spec-interview/<slug>/TRACK.md`. Historical TRACK records may use equivalent headings and wording when their meaning is unambiguous; preserve them, never rewrite them. Every new record uses the canonical field names below, and does not rename, merge, or omit a field its status requires.

```markdown
# TRACK: <slug>

## T5 — Verify persisted behavior  [partial]
Plan version: 2
Covers: FR-008, AC-004
Root: T5
Evidence:
- available automated checks → passed
Verification: unverified
Unresolved:
- Required external observation is unavailable.
Gate: replan required

### Replan checkpoint — T5
Plan version: 2
Blocker: BLK-example-T5
Gate: replan exhausted
root_attempts: 1
continuation_attempts: 0
continuation_limit: 2
total_lineage_attempts: 1
total_lineage_limit: 3

## T6 — Publish authorized release  [blocked]
Plan version: 2
Covers: FR-009, AC-014
Root: T6
Blocker: BLK-example-T6
Blocked because:
- The authorized release target is unavailable to this run.
Resolution condition:
- An authorized release-target check succeeds, or matching access is explicitly attested.
Evidence:
- release-target inspection → no authorized target available
Verification: verified
User action:
- Provide an authorized release target.
Gate: replan required

### Replan checkpoint — T6
Plan version: 2
Blocker: BLK-example-T6
Gate: replan exhausted
root_attempts: 1
continuation_attempts: 0
continuation_limit: 2
total_lineage_attempts: 1
total_lineage_limit: 3
```

The example is one continuous post-task flow, not a collection of alternatives:
T5's first attempt counts as `1/0/1`; exhausting Root T5 does not close Root T6;
the blocked T6 record is immediately followed by its own Full exhaustion
checkpoint; only after both closures may their PLAN be empty and terminal.

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

#### Choose exactly one transition

The resulting `Gate`, not the status alone, selects the transition. `partial`,
`blocked`, and `failed` require `replan required`. `done` and `no_op` normally
use `plan holds`, but verified discovery, deviation, constraint, or missing
coverage may require material replan. Record content remains governed only by
the mandatory-content table above.

This is the only transition table:

| Gate in task record | Valid internal Plan receipt | Resulting PLAN | TRACK checkpoint |
|---|---|---|---|
| `plan holds` | `PLAN maintenance complete` | same version; trigger removed | none |
| `replan required` with valid continuation or repaired strategy | `material replan complete` | version `N+1`; new remainder or repaired future strategy | Full `replan done` |
| `replan required` with no valid continuation | `replan exhausted` | version `N`; triggering Root closed; eligible independent Roots preserved | Full `replan exhausted` |

Choose exactly one row. Any other combination is invalid and remains inside the
post-task critical section.

#### Decide the gate from evidence

Run this after **every** task, without exception. It is a checkpoint, not a rewrite: most gates should avoid material replanning, but even `plan holds` still calls Plan for future-only maintenance.

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

Otherwise set `Gate: plan holds`; Plan performs future-only maintenance before Execute reconciles and returns to root.

**Unconfirmed discoveries never redraw the strategy.** If a `[reported, unconfirmed]` fact would invalidate later tasks, confirm it first. A validation task is valid only when the capability required to perform that validation is observably available now. If a partial task's exact required verification capability is observably unavailable, do not create an unverifiable validation task: the planner must exhaust that same `Root:` and episode. The strategy rewrite waits for verified evidence.

**Mid-task trigger.** Do not wait for the task to end when the divergence is already fatal. If a performer reports `blocked` or `failed` with a reason that invalidates the plan, replan immediately rather than dispatching the next task into a plan you know is wrong.

#### Complete the post-task critical section

`Gate: replan required` is an intermediate state, never a terminal state.

While it is the effective gate for a task, it is forbidden to:

- select or dispatch another task;
- invoke `references/report.md`;
- classify the run as terminal;
- return control to the composite root.

After every task record is appended:

1. Decide the inline gate from verified evidence.
2. Invoke `references/plan.md` with the complete SPEC, current PLAN, complete TRACK, observable state, triggering task, and decided gate. Execute never edits PLAN directly.
3. Match exactly one row of the transition table to exactly one internal Plan receipt. Reject a receipt that does not match the gate.
4. Validate the resulting PLAN with Plan's quality gate. For a `partial` trigger, require exclusively either one direct continuation at the trigger's former relative position with the same `Root:` and `Continues: <trigger task ID>`, or same-Root exhaustion with no continuation. Reject both, neither, a new Root, an unrelated replacement outcome, or a validation task whose required capability is unavailable.
5. Derive `root_attempts`, `continuation_attempts`, and `total_lineage_attempts` from the complete TRACK entries for the triggering Root's current episode. Do not copy counters from Plan, infer them from exhaustion, or use the lineage limits as the observed counts.

##### The only Full checkpoint templates

For a valid material replan, copy the receipt's validated trigger, versions, and
Root and append exactly:

```markdown
### Replan checkpoint — <trigger task ID>
Previous plan version: <N>
New plan version: <N+1>
Gate: replan done (plan version <N+1>)
root_attempts: 1
continuation_attempts: <0-2>
continuation_limit: 2
total_lineage_attempts: <1-3>
total_lineage_limit: 3
```

For valid exhaustion, copy the receipt's validated trigger, version, Root, and
blocker and append exactly:

```markdown
### Replan checkpoint — <trigger task ID>
Plan version: <N>
Blocker: BLK-<slug>-<root-task-id>
Gate: replan exhausted
root_attempts: 1
continuation_attempts: <0-2>
continuation_limit: 2
total_lineage_attempts: <1-3>
total_lineage_limit: 3
```

For `PLAN maintenance complete`, append no checkpoint. If any receipt, PLAN, or
checkpoint is invalid, invoke planning again. Never repair PLAN in Execute and
never append a checkpoint from an invalid receipt.

Re-read complete PLAN and complete TRACK, then evaluate the single exit
predicate below. Do not select another task in this invocation.

A `partial`, `blocked`, or `failed` task with `Gate: plan holds` is a format error. A `done` or `no_op` task may still require material replanning when a verified discovery, deviation, constraint, or coverage gap invalidates the remaining strategy.

`POST-TASK CLOSED(trigger)` is true only when all twelve conditions hold:

1. `trigger` has exactly one task record in TRACK.
2. `trigger` is absent from PLAN.
3. No task ID already present in TRACK remains in actionable PLAN.
4. No PLAN dependency points to a task ID already present in TRACK.
5. PLAN contains exactly one `Plan version:` field.
6. `Status: no-op` is absent whenever TRACK contains any task record.
7. The effective gate of `trigger` is not `replan required`.
8. If material replan occurred, exactly one later Full `replan done` checkpoint names `trigger`, declares previous `N` and new `N+1`, agrees with PLAN, and contains all five attempt fields.
9. If exhaustion occurred, exactly one later Full `replan exhausted` checkpoint names `trigger`, its unchanged `Root:` and Root-derived `Blocker:`, agrees with PLAN, contains all five attempt fields, and leaves no same-Root continuation or task dependent on that Root's unsatisfied state; eligible independent Roots may remain.
10. The checkpoint counters equal attempts actually recorded in the triggering Root's current episode; lineage, episode, and attempt limits remain valid.
11. No prohibited continuation or dependent task was dispatched; surviving task identity and relative order are unchanged, and dependencies remain ordered and acyclic.
12. SPEC, PLAN, TRACK, observable state, lineage fields, blocker cause, resolution condition, coverage, and remaining future work agree.

This check is transient. Do not append it or its verdict to TRACK.

If all twelve pass: return to the composite root. If any one fails: remain in
reconciliation. These are the only two outcomes of the predicate.

### Reopening reconciliation

When the root routes an exhausted lineage back to Execute because its stable blocker now has verified or explicitly attested resolution, reconcile without dispatching a product task:

1. Confirm the named exhaustion checkpoint identifies the trigger and its Root-derived blocker.
2. Append matching resolution evidence without changing the blocker, cause, condition, or history.
3. Invoke Plan under its reopening contract. Reopening creates no inline Gate.
4. Validate the returned PLAN with Plan's quality gate and the reopening contract. Additionally require version +1 and a first replacement with a new numeric ID, the same `Root:`, `Reopens: <exhausted task ID>`, and no `Continues:`.
5. Append the existing named `replan reopened (plan version N)` checkpoint for that exhausted trigger and blocker.
6. Re-read PLAN and TRACK, confirm the reopening checkpoint and future-only PLAN agree, and return to the composite root without dispatching the reopened task.

Reopening changes neither the canonical gate vocabulary nor TRACK's required task fields.

### 6. Exit

After reconciliation, return to the composite root. The root alone decides whether the next reference is Execute or Report under `SKILL.md`; Execute never selects another task, invokes Report, closes `state.md`, or emits the final response.

An effective `replan required`, eligible future PLAN work, or invalid exhaustion state forbids a terminal route. An unresolved exhausted lineage does not forbid independent future work; it reaches the terminal route only after every other eligible lineage is also reconciled and PLAN is empty. On a valid terminal route, Report persists `REPORT.md`; the root then closes `state.md` and emits the persisted body byte-for-byte.

Execution does not construct or paraphrase the terminal response. Invoke `references/report.md` through the existing composite route. After REPORT is persisted and the composite root closes `state.md`, emit exactly the persisted report body byte-for-byte. Do not reconstruct Markdown, add narration, or replace the persisted body.

---

## Escalation

Stop and ask the user — do not decide alone — when:

- an acceptance criterion has become unreachable;
- an action would be destructive or irreversible and the user has not explicitly asked for it;
- the work has drifted far enough from the contract that finishing it would satisfy a different goal;
- a blocker needs access, credentials, or a decision only the user has.

Return the state to the composite root for routing through `references/report.md` rather than improvising a new objective.

---

## Resuming

`TRACK.md` is the resume point. On restart, read the contract, complete `PLAN.md`, and complete `TRACK.md`. First reconcile each interruption window without changing any existing entry: execution-to-TRACK uses the step 2 pre-dispatch state check before recording; TRACK-to-gate resolves the recorded task's effective gate; gate-to-PLAN compares that task's gate and checkpoint to PLAN before any continuation is created. If a task still has effective `replan required` but PLAN is already at a valid higher version, append the canonical named `Replan checkpoint` for that same task with its previous and new versions. If PLAN remains at the same version, invoke the replanner. Never dispatch a recorded task ID.

For every checkpoint that says `replan exhausted`, confirm that it names the triggering task, carries `Blocker: BLK-<slug>-<root-task-id>` derived from that task's `Root:`, and leaves no same-Root continuation or dependent task in the closed episode. Later tasks under other Roots are valid when their prerequisites are independently satisfied. When matching resolution exists, follow Reopening reconciliation above: append the evidence, invoke Plan, validate the new `Reopens:` PLAN, append the named reopening checkpoint, and return to root without dispatching. Preserve all earlier entries and retain the exhausted lineage as historical evidence.

Do not re-run completed tasks. Re-verify a completed task only when a later discovery may have invalidated its postcondition.

The same completion barrier applies after a restart.

After repairing execution-to-TRACK, TRACK-to-gate, or gate-to-PLAN, re-read TRACK and PLAN and run the reconciliation check above. Do not select work or report merely because the interrupted task already has a TRACK entry.

A higher PLAN version closes `replan required` only after the corresponding named `replan done` checkpoint for the same task is appended. The same PLAN version with `replan required` still invokes the replanner. `replan exhausted` keeps its Root and episode closed while its Root-derived blocker remains unresolved; it does not close an independent Root or make the run terminal while eligible future work remains.

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

A lineage permits one root attempt and at most two continuation attempts per episode, for a maximum of three attempts. T1 is the root, T2 and T3 are continuations, and T4 is forbidden in the same episode. Its checkpoints expose `root_attempts: 1`, `continuation_attempts: 0–2`, `continuation_limit: 2`, `total_lineage_attempts: 1–3`, and `total_lineage_limit: 3`; the observed counters are derived from TRACK, never from the limits or from the fact of exhaustion. A blocked or failed lineage records and reuses `Blocker: BLK-<slug>-<root-task-id>` under the same `Root:`. Exhaustion closes only that Root and episode: other Roots remain eligible when they neither continue nor depend on its unsatisfied state. An exhausted episode may reopen only after ordinary inspection verifies, or the user explicitly attests, resolution of that same blocker. Append the resolution evidence, invoke Plan to write the new-version `Reopens:` episode, validate it, then append the named `replan reopened (plan version N)` checkpoint; preserve all prior history. Before any new work reconcile each window: execution-to-TRACK by inspecting side effects before recording, TRACK-to-gate by appending a missing checkpoint without redispatch, and gate-to-PLAN by comparing plan versions and gates before creating a continuation. Never repeat a recorded task ID or a completed side effect.
