# Execute and replan

This is an internal composite stage. Read this entire reference before applying it. Return to the local composite root immediately after it and continue without a user-facing stage handoff. Use only ordinary artifact inspection—files, contents, timestamps, task IDs, and checkpoints—never an executable detector or an external skill.

# Execute Plan

Carry out a plan one task at a time, keeping an honest record of what actually became true.

You are the **orchestrator**. You dispatch each task, verify the result against observable state, write the track, and decide whether the plan still holds. Verification and the track are always yours, whoever performs the work.

This skill is domain-neutral: it executes code work, research, writing, operational, and physical-world plans alike.

## Working directory

```
spec-interview/<slug>/
  SPEC.md      the contract      (read-only here; may be absent â€” then PLAN.md is the contract)
  PLAN.md      the strategy      (read; replaced by references/plan.md)
  TRACK.md    the record        (you are the only writer)
  REPORT.md    written by references/report.md
```

## Execution modes

Pick per task, in this order:

**Delegated (default).** One subagent per task. Costs more total tokens â€” the subagent re-reads context you already hold â€” and buys two things worth more: your context stays clean, so a long run never hits lossy compaction; and the track becomes the only memory channel between tasks, which is what keeps it honest. A track written inline duplicates your context and quietly rots.

**Inline.** When the host provides no subagents, when the user asks for it, or when the whole plan is two trivial tasks. Everything else in this skill is unchanged â€” including verifying the postcondition as a separate act from doing the work.

**Human handoff.** When the task must be performed by a person or outside your reach â€” a physical action, an approval, an access grant. Present the task, its `Done when`, and what evidence you need in plain language; never hand a person the structured return contract below. Ask only: did it happen, what is now true, and anything unexpected. **You** turn that answer into the track entry, recording it as `attested`. Never mark such a task done on the assumption it happened.

---

## The loop

```
read contract + PLAN + TRACK
   â”‚
   â–¼
select next task â”€â”€â”€â”€ none left â”€â”€â”€â”€â–º invoke references/report.md
   â”‚
   â–¼
does its postcondition already hold? â”€â”€ yes â”€â”€â”
   â”‚ no                                       â”‚
   â–¼                                          â”‚
dispatch (delegated / inline / human)         â”‚
   â”‚                                          â”‚
   â–¼                                          â”‚
verify against observable state               â”‚   â—„â”€â”€ you do this, not the performer
   â”‚                                          â”‚
   â–¼                                          â–¼
append track entry â—„â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
   â”‚
   â–¼
replan gate â”€â”€â”€â”€ plan still true â”€â”€â”€â”€â–º next task
   â”‚
   â””â”€â”€ plan invalid â”€â”€â–º invoke references/plan.md in replan mode

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

- Nothing ever attempted this task â€” it was already true â†’ `no_op`.
- A previous run may have acted and died before recording â†’ `done`, with the observed state delta and a note that it was reconstructed after an interruption. Recording that as `no_op` would erase a real change from the record.

When you cannot tell the two apart, assume the second. Losing a state delta is worse than over-reporting one.

### 3. Dispatch

The performer starts cold, so the brief must stand alone. Send exactly this:

```markdown
## Task
<the full task block from PLAN.md â€” T-id, Task, Done when, Verify by, Covers>

## Goal context
<the one-sentence goal from PLAN.md>

## Constraints that apply
<constraints, business rules, and non-goals from the contract that bear on this task â€”
not the whole contract>

## Established facts
<the [verified] Discovered entries from TRACK.md that this task depends on â€”
identities, paths, versions, decisions already made â€” plus any fact from the task's
Reasoning the performer needs; for T1, the grounding observation. A fact still
[reported, unconfirmed] is not established: confirm it first, or pass it explicitly
labeled as an unconfirmed report>

## Return contract
Do the work, then verify your own postcondition and return ONLY this block:

status: done | partial | blocked | failed | no_op
state_delta:
- <what is observably different now, and its consequence>
evidence:
- <check performed> â†’ <result>
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
- Report state, not activity. "Requests over 30s now fail" â€” not "I edited the client".
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

The brief deliberately omits the task's `Reasoning` â€” that is the planner's justification, not an instruction. If the reasoning contains a fact the performer needs, that fact belongs in `Established facts`.

### 4. Verify â€” do not take the performer's word

The performer is not the judge of its own success. Before writing the track, check the postcondition yourself against observable state:

- Run the `Verify by` check, or confirm the reported evidence is real and current.
- Confirm the `Done when` condition actually holds now.
- If `done` was claimed but the postcondition does not hold, record `failed` with the discrepancy. Do not soften it.

Extend the same skepticism to `discovered:` items â€” they become later tasks' input and the replanner's premises. Confirm each one against observable state when a cheap check exists and record it `[verified]`; when you cannot, record it `[reported, unconfirmed]`. An unconfirmed discovery is a hypothesis, not a fact.

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

## T1 â€” Payment client timeout  [done]
Plan version: 1
Covers: FR-004, AC-002
State delta:
- Outbound payment requests now fail after 30 seconds instead of hanging indefinitely.
Evidence:
- integration test PaymentTimeoutTests â†’ passed
Verification: verified
Gate: plan holds

## T2 â€” Product endpoint cache  [partial]
Plan version: 1
Covers: FR-007, AC-005
State delta:
- The cache abstraction exists and the product endpoint reads through it.
Evidence:
- unit tests â†’ passed (14)
- production Redis behavior â†’ not checked, no access to that environment
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
```

TRACK rules:

- **Semantic content is mandatory; canonical decoration is not.** Every task record contains task identity, status, plan version, coverage, state delta, evidence, verification, and every applicable discovery, unresolved item, risk, user action, or deviation. Associate each task with exactly one Gate either in that task record or in a subsequent checkpoint that names the task. `### Gate checkpoint — <task ID>` is recommended style, not a semantic Must.

- **Append every task entry and every gate checkpoint.** `TRACK.md` is append-only: never edit a prior task or checkpoint. After each task entry append `### Gate checkpoint — <task ID>` with plan version, evidence, and one Gate: `plan holds`, `replan required`, `replan done (plan version N)`, or `replan exhausted`. A later fact is a correction checkpoint naming the affected task; history is never rewritten.
- **On a run with no `Covers`** â€” no SPEC, so the contract is the union of the plan's `Done when` â€” copy the task's `Done when` into the entry instead, along with its `Restates:` line when present. Completed tasks leave the plan, so this is the only place the satisfied part of the contract survives.

- **Write it after every task, never reconstruct it at the end.** A track rebuilt from memory at the end of a long run is exactly the semantic loss this workflow exists to prevent.
- **Copy `Covers` verbatim from the plan.** It is the only path from task evidence back to acceptance criteria once the task leaves the plan. Omit when the plan has no `Covers`.
- **Every `Discovered` item carries its provenance label** â€” `[verified]` when you confirmed it against observable state, `[reported, unconfirmed]` when it is only the performer's claim. Briefs, replans, and the gate treat the two differently, so an unlabeled discovery is a format error.
- **State over activity.** "Authentication now rejects expired tokens" â€” not "edited AuthService". Filenames are optional traceability, appended after the consequence.
- **Evidence, not reasoning.** Record what was checked and what it returned. Do not record deliberation.
- **Record deviations honestly.** A silent deviation becomes a hidden contract change.
- **Destructive or irreversible actions and changes to user data are always state deltas.** Never leave one implicit.
- A recovered transient error with no residual consequence may be omitted. An error that changed the final state must be recorded.
- Append only. A later fact is a new correction checkpoint naming the earlier task; never rewrite history.

### 6. Status transitions

**This table is normative.** Where the diagram, the prose, or the anti-patterns seem to say otherwise, the table wins. Every status has exactly one continuation, and no status leaves the loop undefined.

| Status | TRACK | Then |
|---|---|---|
| `done` | record with verification label | replan gate â†’ next task |
| `no_op` | record with the evidence that it was already true | replan gate â†’ next task |
| `partial` | record, with `Unresolved` filled | replan gate **must** run â€” the remainder needs a new task |
| `blocked` | record, with the blocker and any `user_action` | replan gate **must** run; if it cannot route around it â†’ report |
| `failed` | record, with the discrepancy | replan gate **must** run |

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

### 8. Exit

Stop the loop and invoke `references/report.md` when:

- every task is `done` or `no_op`; or
- the plan has zero tasks because the goal was already satisfied; or
- a task is `blocked` and replanning cannot route around it; or
- the plan cannot be repaired without changing the contract; or
- the user asks to stop.

Never stop silently. Every exit goes through the reporter.

---

## Escalation

Stop and ask the user â€” do not decide alone â€” when:

- an acceptance criterion has become unreachable;
- an action would be destructive or irreversible and the user has not explicitly asked for it;
- the work has drifted far enough from the contract that finishing it would satisfy a different goal;
- a blocker needs access, credentials, or a decision only the user has.

Report the state through `references/report.md` rather than improvising a new objective.

---

## Resuming

`TRACK.md` is the resume point. On restart, read the contract, `PLAN.md`, and `TRACK.md`. First reconcile each interruption window without changing any existing entry: execution-to-TRACK uses the step 2 pre-dispatch state check before recording; TRACK-to-gate appends a missing checkpoint for the recorded task; gate-to-PLAN compares the last checkpoint's plan version and gate before any continuation is created. If `replan required` has a higher `PLAN.md` version, append `replan done (plan version N)`; at the same version invoke the replanner. Never dispatch a recorded task ID.

If the last checkpoint says `replan exhausted`, it is terminal until resolution of its `Blocker: BLK-<slug>-<root-task-id>` is verified by ordinary inspection or explicitly attested by the user. The evidence must match that stable blocker identity. Append the resolution evidence and a reopening checkpoint, invoke the replanner, and start a same-SPEC new `Reopens:` episode; preserve all earlier entries and retain the exhausted lineage as historical evidence.

Do not re-run completed tasks. Re-verify a completed task only when a later discovery may have invalidated its postcondition.

---

## Anti-patterns

**Trusting the return.** Accepting `status: done` without checking the postcondition. The performer is optimistic about its own work; that is the whole reason for the verification step.

**Redispatching unfinished tasks.** Re-running a `partial` task from the top repeats its side effects. The remainder is new work and needs a new task.

**Batch track writing.** Running five tasks and then writing five entries from memory. The details that mattered are already gone.

**Activity tracks.** Entries that list what was touched instead of what became true. The report cannot be built from those.

**Skipping the replan gate when things are going well.** The plan is most often wrong exactly when execution feels smooth â€” because nothing has forced you to look at it.

**Fixing the plan inline.** When the plan is wrong, invoke the replanner. Silently doing something other than what the plan says produces a run nobody can audit.

**Reinventing the todo list.** `PLAN.md` is the durable artifact. Any ephemeral task-tracking UI is a view of it, never a second source of truth.


## Full protocol

Every replan continuation has `Continues: <root task ID>`.

Full includes Bounded's root T1 plus only continuation T2/T3 limit; T4 is forbidden. Its checkpoints expose `root_attempts: 1`, `continuation_attempts: 0–2`, `continuation_limit: 2`, `total_lineage_attempts: 1–3`, and `total_lineage_limit: 3`. A blocked or failed lineage records and reuses `Blocker: BLK-<slug>-<root-task-id>`. An exhausted episode may reopen only after ordinary inspection verifies, or the user explicitly attests, resolution of that same blocker. Append the evidence and a `replan reopened (plan version N)` checkpoint, preserve prior history, and start a new `Reopens:` episode rather than extending the exhausted lineage. Before any new work reconcile each window: execution-to-TRACK by inspecting side effects before recording, TRACK-to-gate by appending a missing checkpoint without redispatch, and gate-to-PLAN by comparing plan versions and gates before creating a continuation. Never repeat a recorded task ID or a completed side effect.
