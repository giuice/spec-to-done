# Execute and replan

This is an internal composite stage. Read this entire reference before applying it. Return to the local composite root immediately after it and continue without a user-facing stage handoff. Use only ordinary artifact inspection—files, contents, timestamps, task IDs, and checkpoints—never an executable detector or an external skill.

# Execute Plan

Carry out a plan one task at a time, keeping an honest record of what actually became true.

You are the **orchestrator**. You dispatch each task, verify the result against observable state, write the track, and decide whether the plan still holds. Verification and the track are always yours, whoever performs the work.

This skill is domain-neutral: it executes code work, research, writing, operational, and physical-world plans alike.

## Working directory

```
spec-interview/<slug>/
  SPEC.md      the contract      (mandatory; read-only here)
  PLAN.md      the strategy      (read; replaced by references/plan.md)
  TRACK.md    the record        (you are the only writer; active segment)
  track/      sealed segments   (read only for a specific entry)
  SNAPSHOT.md current state     (you are the only writer; rewritten at every gate)
  REPORT.md    written by references/report.md
```

`SPEC.md` is mandatory. If it is absent, malformed, or its readiness is not recorded in `state.md` as `Verdict: Ready`, execution does not begin: do not dispatch a task, do not verify one, and do not replan. Preserve every existing artifact untouched and return control to the composite root for Specify. A PLAN or a TRACK is evidence of what was attempted, never a substitute contract.

## Execution modes

Pick per task, in this order:

**Delegated (default).** One subagent per task. Costs more total tokens — the subagent re-reads context you already hold — and buys two things worth more: your context stays clean, so a long run never hits lossy compaction; and the track becomes the only memory channel between tasks, which is what keeps it honest. A track written inline duplicates your context and quietly rots.

**Inline.** When the host provides no subagents, when the user asks for it, or when the whole plan is two trivial tasks. Everything else in this skill is unchanged — including verifying the postcondition as a separate act from doing the work.

**Human handoff.** When the task must be performed by a person or outside your reach — a physical action, an approval, an access grant. Present the task, its `Done when`, and what evidence you need in plain language; never hand a person the structured return contract below. Ask only: did it happen, what is now true, and anything unexpected. **You** turn that answer into the track entry, recording it as `attested`. Never mark such a task done on the assumption it happened.

---

## The loop

With no execution history in active or sealed TRACK segments, skip recovery and initial SNAPSHOT persistence: select the first task and create the first snapshot only after its task checkpoint and PLAN are reconciled. The pre-dispatch side-effect check still applies.

```
read contract + PLAN + available SNAPSHOT/TRACK
   +-- no execution history --> select next task
   |
   v
reconcile interruptions (sealed entries only as needed; see Resuming)
   |
   v
seal if due; persist current SNAPSHOT
   |
   v
select next task ---- none left ----> invoke references/report.md
   |
   v
does its postcondition already hold? -- yes --+
   | no                                       |
   v                                          |
dispatch (delegated / inline / human)         |
   |                                          |
   v                                          |
verify against observable state               |   <-- you do this, not the performer
   |                                          |
   v                                          v
append track entry <-------------------------+
   |
   v
replan gate (with ID reservation)
   +-- plan holds --> references/plan.md: maintenance --+
   +-- invalid ----> references/plan.md: replan --------+--> reconcile gate/PLAN
   +-- terminal --------------------------------------+          |
                                                                 v
                                          seal if due; persist SNAPSHOT
                                                                 |
                                                                 v
                                                    next task or report

Every path reaches the track and then the gate. No status skips either.
```

### 1. Select the next task

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
- Do not claim a check you did not run.
- A `discovered` item names what you observed and where, so it can be confirmed.
- Use `no_op` if the postcondition was already satisfied before you started.
- If the task is impossible as written, return `blocked` or `failed` with the reason.
  Do not improvise a different task.
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

### 5. Write the track entry

**You are the only writer.** Performers return structured blocks; you validate and append. One writer means one format, no concurrent writes, and a natural point to decide on replanning.

Append to `spec-interview/<slug>/TRACK.md`. The rendering below is preferred human-readable style, not a separate state protocol: equivalent headings and wording are valid when the durable meaning is complete and every task can be associated unambiguously with its gate.

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
Highest task ID reserved: T2
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
Highest task ID reserved: T2
Gate: replan required

### Replan checkpoint — T2
Previous plan version: 1
New plan version: 2
Highest task ID reserved: T3
Gate: replan done
```

TRACK rules:

- **Semantic content is mandatory; canonical decoration is not.** Every task record contains task identity, status, plan version, coverage, lineage, state delta, evidence, verification, and every applicable discovery, unresolved item, risk, user action, or deviation. Equivalent headings and wording are valid wherever the durable meaning is complete.

- **The initial gate is inline: every task record ends with its own `Gate:` line,** holding `plan holds` or `replan required`. Writing it inside the record is what makes the association positional, so it can never be read as belonging to another task.

- **Every later gate transition is a separate checkpoint that names the task,** carrying the plan version and the new value — `replan done (plan version N)` or `replan exhausted`. The current gate for a task is the last gate event associated with it.

- **A gate that names no task is a format error.** One gate covering several tasks at once, or a trailing global gate at the end of the file, destroys the task-to-gate association the report is built from. It is not untidy style; it is unusable evidence.

- **Append every task entry and every gate checkpoint.** `TRACK.md` is append-only: never edit a prior task or checkpoint, including to revise a gate. A gate changes by appending a new checkpoint, never by rewriting the old value. A later fact is a correction checkpoint naming the affected task; history is never rewritten.
- **Write it after every task, never reconstruct it at the end.** A track rebuilt from memory at the end of a long run is exactly the semantic loss this workflow exists to prevent.
- **Copy `Root`, and any `Continues` or `Reopens`, verbatim from the plan.** They are the lineage record; without them TRACK cannot show which attempt followed which, and the blocker loses its stable identity.
- **Copy `Covers` verbatim from the plan.** It is the only path from task evidence back to acceptance criteria once the task leaves the plan. Every task has one; a task without it is a plan defect, not a run to continue.
- **Every `Discovered` item carries its provenance label** — `[verified]` when you confirmed it against observable state, `[reported, unconfirmed]` when it is only the performer's claim. Briefs, replans, and the gate treat the two differently, so an unlabeled discovery is a format error.
- **State over activity.** "Authentication now rejects expired tokens" — not "edited AuthService". Filenames are optional traceability, appended after the consequence.
- **Evidence, not reasoning.** Record what was checked and what it returned. Do not record deliberation.
- **Record deviations honestly.** A silent deviation becomes a hidden contract change.
- **Destructive or irreversible actions and changes to user data are always state deltas.** Never leave one implicit.
- A recovered transient error with no residual consequence may be omitted. An error that changed the final state must be recorded.
- Append only. A later fact is a new correction checkpoint naming the earlier task; never rewrite history.

### 6. Status transitions

**This table is normative.** Where the diagram, the prose, or the anti-patterns seem to say otherwise, the table wins. Every status has exactly one continuation, and no status leaves the loop undefined.

| Status | TRACK | Then |
|---|---|---|
| `done` | record with verification label | replan gate → maintain or replan → finish checkpoint |
| `no_op` | record with the evidence that it was already true | replan gate → maintain or replan → finish checkpoint |
| `partial` | record, with `Unresolved` filled | replan gate **must** run — the remainder needs a new task |
| `blocked` | record, with the blocker and any `user_action` | replan gate **must** run; if it cannot route around it → finish checkpoint → report |
| `failed` | record, with the discrepancy | replan gate **must** run |

Every gate route finishes its checkpoint before the next task or report: reconcile gate/PLAN, seal if due, then persist SNAPSHOT. Partial and failed tasks require replanning or an honest terminal route, never redispatch.

### 7. Replan gate

Run this after **every** task, without exception. It is a checkpoint, not a rewrite: most gates should pass without a replan.

Include `Highest task ID reserved` with the task's inline gate (or named recovery checkpoint) before PLAN maintenance or replanning: retain the maximum from PLAN metadata, its task IDs, and the previous reservation checkpoint. Recover missing reservation metadata as specified in `references/plan.md`.

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

Otherwise set `Gate: plan holds` and invoke `references/plan.md` for future-only maintenance before continuing. Maintenance removes the completed task from PLAN, keeps the plan version unchanged, and appends no checkpoint. A PLAN that still lists an attempted or completed task ID is not a plan that holds.

After replanning, append the task-named outcome checkpoint with the resulting plan version and ID reservation. Finish the checkpoint as defined above; do not dispatch between gate/PLAN reconciliation and SNAPSHOT persistence.

**Unconfirmed discoveries never redraw the strategy.** If a `[reported, unconfirmed]` fact would invalidate later tasks, confirm it first; if confirming it takes real work, the only replan it may drive is one that adds a validation task. The strategy rewrite waits for that task's verified result.

**Mid-task trigger.** Do not wait for the task to end when the divergence is already fatal. If a performer reports `blocked` or `failed` with a reason that invalidates the plan, replan immediately rather than dispatching the next task into a plan you know is wrong.

### 8. Exit

Stop the loop and invoke `references/report.md` when:

- every task is `done` or `no_op`; or
- the plan has zero tasks because the goal was already satisfied; or
- a task is `blocked` and replanning cannot route around it; or
- the plan cannot be repaired without changing the contract; or
- the user asks to stop.

Never stop silently. Every exit goes through the reporter.

Before taking a terminal route, confirm PLAN contains no remaining actionable task and no task ID already present in TRACK. Remaining future work forbids ordinary completion; reconcile it or report the run as unfinished.

Execution never closes `state.md` and never authors the final response. Invoke `references/report.md` through the composite route and return to the root, which closes `state.md` after `REPORT.md` is persisted.

---

## Escalation

Stop and ask the user — do not decide alone — when:

- an acceptance criterion has become unreachable;
- an action would be destructive or irreversible and the user has not explicitly asked for it;
- the work has drifted far enough from the contract that finishing it would satisfy a different goal;
- a blocker needs access, credentials, or a decision only the user has.

Report the state through `references/report.md` rather than improvising a new objective.

---

## Track segments and the snapshot

On a long run the record outgrows a single read. Two rules keep resuming cheap without weakening the record.

**Segments.** The execution record is the concatenation, in order, of the sealed segments under `spec-interview/<slug>/track/` and the active `TRACK.md`. Wherever this workflow says TRACK, it means that whole record. Seal the active segment at a phase or milestone boundary of the plan, or whenever it no longer fits in a single read: move it unchanged to `track/TRACK-<nn>-<label>.md` (`nn` is a two-digit sequence, `label` names the phase), then start a new `TRACK.md` whose first entry is a segment checkpoint:

```markdown
### Segment opened — <label>
Sealed: track/TRACK-<nn>-<label>.md
Plan version: N
Highest task ID reserved: T<n>
Open lineages: <root> (<attempts used>/3, Blocker: BLK-<slug>-<root> or none), ...
```

- Seal only after the task's gate and PLAN have been reconciled (maintenance or recorded replan outcome), never while a gate or replan is pending.
- `Segment opened` is not a task gate. To recover the latest task's effective gate, inspect its last associated gate event; if absent from the active file, follow `Sealed` into the preceding segment, continuing backward as needed. No global gate is introduced.
- Sealing moves bytes; it never rewrites, reorders, or summarizes them. A sealed segment is never edited again.
- Task IDs stay unique across all segments.
- Read a sealed segment only for a specific entry you need; do not reread the whole record to resume.

**Snapshot.** `SNAPSHOT.md` is a short view of the current state. Persist it after reconciling the task record, gate, and PLAN, and after any sealing — before the next dispatch or report. It is the one execution artifact replaced rather than appended:

```markdown
# SNAPSHOT: <slug>
Updated after: <TRACK file + entry heading> (plan version N)

## Deliverables
- <path> — <current version or state> — <accepted | in review | rework | not started>

## Open lineages
- Root <id>: attempt <n>/3, last status <status>, blocker <BLK-... | none>

## Blockers
- BLK-<slug>-<root>: <what is blocked> — resolution task <id> — resolution verified: yes | no

## Constraints and decisions affecting remaining work
- <operational consequence> — affects <task or criterion IDs> — source <TRACK file + entry heading>

## Unresolved
- <gap, risk, unverified assumption, or pending check> — affects <criterion IDs> — source <TRACK file + entry heading>

## Coverage pointers
- <criterion IDs> — <TRACK file + entry heading containing evidence and any later correction>
```

- SNAPSHOT is derived. TRACK wins every disagreement; SNAPSHOT holds no evidence and no history, only the current state and pointers to the TRACK entries that record it.
- Read an available snapshot as an aid during reconciliation; never persist it instead of, or ahead of, the task record, gate, and PLAN reconciliation.
- Update from the last valid snapshot plus subsequent TRACK entries; reread older entries only to resolve a specific gap or conflict. A snapshot is stale when `Updated after` differs from the last TRACK entry or current PLAN version. Reconcile it before dispatch; rebuild from the whole record only if missing, inconsistent, or its source checkpoint cannot be found. After sealing, update pointers to the sealed file.
- At each gate, retain decisions that constrain remaining tasks or verification, even if already implemented. Remove one only with a TRACK-recorded reason: superseded, condition verified and no longer needed, no remaining task or check affected, or preserved in a document the workflow must read before acting. Keep uncertain cases; never remove their history.
- Carry unresolved items until TRACK explicitly resolves or supersedes them; a phase boundary or lack of a blocker is not resolution. Keep unverified assumptions labelled, never promote them to facts.
- Aim for one page: one line per item, omit empty sections, reference SPEC and PLAN instead of duplicating them, and leave rationale and evidence in TRACK. Group coverage pointers where possible; preserve material state even when it exceeds one page. Pointers locate evidence; they do not establish satisfaction.

**Migration.** A TRACK written before this rule has no segments. After its next reconciled gate/PLAN, seal it unchanged (phase boundaries are optional); open a new segment with the checkpoint above and build `SNAPSHOT.md` from the whole record once.

---

## Resuming

When execution history exists, read the contract, PLAN, available SNAPSHOT, and active TRACK; consult sealed entries only as needed. Reconcile before persisting the snapshot:

1. **Interrupted sealing:** if TRACK is absent after the move, locate the latest numbered sealed segment and create the opening entry from it and PLAN. If an opening entry is incomplete, preserve it and append a complete one. Do not move the sealed record again. Ambiguous or conflicting files require resolution, not a guessed history.
2. **Task/gate/PLAN:** locate the latest task and its effective gate, ignoring segment-opening markers. Execution-to-TRACK uses step 2's side-effect check before recording; TRACK-to-gate appends a task-named missing gate with ID reservation, without redispatch. For `plan holds`, finish PLAN maintenance. For `replan required`, if the higher PLAN version already contains that task's valid future-only replan, append its missing `replan done (plan version N)` checkpoint; at the same version invoke the replanner. Preserve IDs, lineage, and all existing entries; conflicting versions require resolution before continuation.
3. **Resume state:** after reconciliation, finish any due sealing, then persist SNAPSHOT using the incremental/full-rebuild distinction above. An outdated pointer to moved TRACK content is resolved in its sealed segment. Never dispatch a recorded task ID.

If the effective task gate says `replan exhausted`, it is terminal until resolution of its `Blocker: BLK-<slug>-<root-task-id>` is verified by ordinary inspection or explicitly attested by the user. The evidence must match that stable blocker identity. Append the resolution evidence and a reopening checkpoint, invoke the replanner, and start a same-SPEC new `Reopens:` episode under the same `Root:`; preserve all earlier entries and retain the exhausted lineage as historical evidence.

Do not re-run completed tasks. Re-verify a completed task only when a later discovery may have invalidated its postcondition.

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
