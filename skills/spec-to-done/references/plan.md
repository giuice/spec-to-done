# Plan From Specification

Produce a high-level, outcome-shaped plan that an executor can carry out, and keep its future strategy true as reality diverges. Planning owns decomposition, ordering, observable postconditions, and strategy repair. It does not execute work, decide what success means, write TRACK, or report the terminal outcome.

The boundary is absolute: **replanning may change how the goal is reached; it never changes what counts as success.**

## Artifacts and inputs

One work item lives in `spec-interview/<slug>/`:

```text
SPEC.md       the contract; mandatory and read-only to planning
PLAN.md       the actionable future strategy; written or maintained here
TRACK.md      append-only execution history; input during maintenance or replan; never written here
REPORT.md     terminal communication; owned by reporting
```

Planning is self-contained. Use only the contract, current PLAN, TRACK, and observable current state available to the run. Do not depend on repository source snapshots, external documentation, scripts, provider-specific APIs, or another runtime skill.

## Operational call contract

For the maintenance or material-replan call made immediately after a newly recorded task, the executor supplies all six inputs together:

```text
- the complete SPEC;
- the complete current PLAN;
- the complete TRACK;
- current observable state;
- the triggering task;
- the executor-decided Gate: plan holds | replan required.
```

Planning reads the complete inputs, writes the resulting PLAN, validates it as future-only, and returns control to the executor. Planning never writes TRACK, never appends a gate checkpoint, and never executes a task. The executor may reject an invalid planner result and call planning again, but it must not repair PLAN itself.

A reopening call uses the same complete SPEC, PLAN, TRACK, and observable state, plus the exhausted trigger and verified or attested resolution of its stable blocker. It does not create or change an inline Gate. Planning writes the first future-only PLAN of the new episode; Execute validates it and appends the existing named `replan reopened` checkpoint.

The planner returns exactly one **internal receipt** to Execute. A receipt is
transient coordination output: it is never written or copied into TRACK, PLAN,
REPORT, or `state.md`; it adds no gate, canonical state, artifact, or required
field. Execute uses it only to validate the PLAN transition and render the
existing checkpoint.

Maintenance and material replan use these receipts:

```text
outcome: PLAN maintenance complete
trigger_task_id: <T-id>
plan_version: <N>

outcome: material replan complete
trigger_task_id: <T-id>
previous_plan_version: <N>
new_plan_version: <N+1>
Root: <root task ID>
```

The exhaustion receipt is defined once beside **Replan exhausted** below.
Return only the applicable internal receipt and no narration. Planning does not
return attempt counters; Execute derives them from the complete TRACK lineage
and copies the validated identity/version values into one checkpoint. Execute
never persists the receipt itself.

## The SPEC is required

Planning operates on a Ready SPEC and nothing else. `spec-interview/<slug>/state.md` must record per-domain coverage and `Verdict: Ready`, which only Specify writes.

If no Ready SPEC exists, do not plan. Preserve every artifact already present, return control to the composite root for Specify, and let it route back here once readiness is recorded. A stated goal, a supplied PRD, a detailed handoff, or an existing PLAN is interview input, never a contract. Never invent a SPEC and never convert an existing PLAN into one.

Every plan therefore names its contract as `Spec: ./SPEC.md`, and every task carries `Covers:`.

## 1. Ground the plan in observation, never assumption

Before writing or regenerating tasks, inspect actual current state. Use what the domain makes observable: files and structure, configuration, running behavior, available tools, data, prior work, external services, or people.

The reasoning for `T1` must cite a concrete observation: what exists, what is missing, and which contract assumptions it confirms or contradicts. Do not infer a premise merely because a goal mentions it.

If observation contradicts the contract, stop and escalate the contradiction. Do not quietly plan around it. If the goal is already true before execution begins, produce no work merely to make a plan look substantial; use the initial no-op form below. Once TRACK contains any task record, `Status: no-op` is forbidden: historical execution must be reconciled and reported from TRACK rather than reclassified as an initially satisfied goal.

## 2. Choose outcome-sized tasks

A task is a meaningful unit of work with one verifiable outcome. It normally groups several low-level actions.

```text
Too fine                              Outcome-sized
Open the config file                  Add Redis connection settings that read
Add a redis section                   host and port from environment variables.
Save the file

Search for papers                     Identify the three most-cited papers on
Open each result                      retrieval-augmented generation published
Note citation counts                  since 2023, with citation counts.
```

```text
Too coarse                            Outcome-sized
Implement caching                     Add a cache abstraction with get, set, and
                                      invalidate operations.
                                      Wire the product endpoint through it.
                                      Add invalidation on product update.
```

- If a task cannot fail in an interesting way, it is too fine; merge it.
- If it has more than one postcondition, it is too coarse; split it.
- Make every task stand alone: it may be handed to a separate agent or person.
- State the intended result and concrete parameters, not mechanics. “The payment client fails a request after 30 seconds” is a plan; “open `PaymentClient.cs` and set `Timeout`” is an action list.
- A genuinely unknown branch may be explicit in plain language, such as “If the migration tool supports rollback, use it; otherwise create a rollback script.” Avoid nested or ambiguous conditions.

## 3. Make every task checkable

Every task has a state-based `Done when` and an evidence-producing `Verify by`.

```markdown
Done when: The second identical product request within the TTL is served from cache.
Verify by: Run the ProductCacheTests integration test.

Done when: Every interviewee has confirmed the scheduled slot in writing.
Verify by: Inspect the six invitations for accepted responses.
```

- `Done when` describes state, not activity. “The cache code is written” is activity; “a repeated request is served from cache” is state.
- `Verify by` names a command, test, file check, API response, document, or direct observation that produces evidence. Prefer a mechanical result. When judgment is necessary, name the artifact and the criterion to inspect.
- Use `Verify by: user confirmation` only when no other observation is possible. Confirmation is valid `attested` evidence once actually given; awaiting it is not a satisfied condition.
- A condition with no possible postcondition is not a task. Merge it into an outcome task or remove it.

## PLAN format

Write the current future-only strategy to `spec-interview/<slug>/PLAN.md`:

```markdown
# PLAN: <slug>

Spec: ./SPEC.md
Goal: <one sentence, restated from the contract or the user's words verbatim>
Plan version: 1
Replanned because: <concrete verified trigger>  # omit on version 1 and maintenance

## T1 — <outcome-shaped title>
Reasoning: <why this is one outcome; T1 includes the concrete observation grounding the plan>
Task: <what must become true, with concrete parameters>
Done when: <observable postcondition>
Verify by: <evidence-producing check>
Covers: FR-001, AC-003
Root: T1
Depends on: —

## T2 — <outcome-shaped title>
Reasoning: <why this future outcome is needed>
Task: <what must become true>
Done when: <observable postcondition>
Verify by: <evidence-producing check>
Covers: FR-002
Root: T2
Depends on: T1
```

Field rules:

- IDs are stable and never reused. A surviving task retains its ID across plan versions; a new task takes the next unused number from PLAN and TRACK. IDs attached to completed or attempted work are historical and cannot be recycled.
- A surviving ID remains bound to the same future outcome. Materially changing `Task`, `Done when`, `Verify by`, `Covers`, lineage, purpose, or the result the task is intended to produce creates a new task and requires a new ID. Wording may be clarified under the same ID only when that identity is unchanged.
- `Covers:` is mandatory on every task and maps to requirement and acceptance-criterion IDs that exist in the SPEC. Every Must-priority requirement and every acceptance criterion must be covered by TRACK `done`/`no_op` entries plus remaining PLAN tasks.
- `Root:` is mandatory on every task and names the origin of its lineage. A task that starts one names itself; a continuation or a reopening carries the root's ID unchanged, however long the lineage grows. It is unconditional so that `TRACK.md` stays append-only: a root entry is written before anyone knows a continuation will exist, and it can never be edited afterwards to add the field.
- `Continues:` appears only on a replan task that plans the remainder of one `partial`, `blocked`, or `failed` task, and names that **immediate** attempted predecessor, not the root.
- `Reopens:` replaces `Continues:` on the first replacement after a confirmed reopening from `replan exhausted`. It names the last exhausted attempt and keeps the same `Root:`, which preserves history while starting a new attempt episode.
- The stable blocker identity is derived from `Root:`, never from the immediate predecessor, so one lineage keeps one blocker across every attempt.

```markdown
## T1        Root: T1                      the lineage starts
## T2        Continues: T1    Root: T1     T1 came back partial
## T3        Continues: T2    Root: T1     T2 came back partial
## T4        Reopens: T3      Root: T1     the episode was exhausted and reopened
```

- `Depends on:` lists prerequisite task IDs; use `—` when none. Place every prerequisite before its dependent and keep the dependency graph acyclic.
- `Plan version` labels the strategy used for TRACK entries. PLAN is not an archive: it contains only actionable future work. TRACK preserves completed and attempted history.

## Initial planning

Use initial planning when a contract or checkable goal exists and no actionable PLAN exists.

When observation proves the goal was already true and TRACK contains zero task records, write this zero-task plan and return control to the composite root for reporting:

```markdown
# PLAN: <slug>

Spec: ./SPEC.md
Goal: <one sentence>
Plan version: 1
Status: no-op
Already true because: <the observation that establishes it>
Evidence: <how it was observed>
```

`Status: no-op` is legal only in this initial, zero-task, zero-TRACK-record case. A PLAN with tasks or any PLAN produced after a TRACK task record omits it. Maintenance, material replan, exhaustion, and reopening must reject it rather than use it as a terminal shortcut.

1. Read the SPEC completely.
2. Observe current state and stop for a contradiction or write the zero-task no-op plan if it is already true.
3. Decompose only the remaining work into outcome-sized, checkable tasks.
4. Map all Must requirements and acceptance criteria to TRACK history plus future tasks.
5. Run the quality gate below, fix every failure, and write version 1.

Planning stops at the written strategy and returns control to the composite root. It never performs planned work.

## Checkpoint maintenance after `plan holds`

Use this mode only when the executor supplies `Gate: plan holds`. Verified feedback has not changed the remaining strategy, so maintenance removes history from the future plan without redesigning any outcome.

1. Remove from PLAN every task ID already present in TRACK.
2. Preserve the current version exactly and keep exactly one `Plan version:` field.
3. Preserve each surviving task's ID, outcome, `Task`, `Done when`, `Verify by`, `Covers`, lineage, purpose, and relative order.
4. Preserve every dependency between surviving tasks exactly. For a dependency that names an ID in TRACK, remove it when that task's `done`/`no_op` evidence satisfies the prerequisite; otherwise repoint it only to an unchanged surviving task that already represents the remaining prerequisite without changing either task's outcome identity. If neither is valid, maintenance is invalid: return control without writing PLAN so Execute remains inside reconciliation; do not invent a prerequisite.
5. Do not add `Replanned because:`, `Continues:`, or `Reopens:`.
6. Do not create a checkpoint. Planning writes PLAN; the executor owns TRACK and appends checkpoints only when the gate requires one.
7. Run the PLAN quality gate and return a validated future-only PLAN.

This deterministic maintenance is required even when no future work remains. A completed or no-op task must not remain in PLAN, and unchanged feedback is not a pretext for version churn, new IDs, or changed survivor identity.

## Replanning only after material divergence

Enter replan mode only from the execution checkpoint when verified feedback makes future strategy false or incomplete, or when a `replan exhausted` run is reopened after the executor verifies or obtains user attestation that its blocker is resolved. Merely reaching the end of a successful task is not a replan trigger.

Use the operational call contract above: immutable SPEC, complete current PLAN, complete append-only TRACK, current observable state, triggering task, and the executor-decided `Gate: replan required`.

### Reflect first

Start the first future task’s reasoning with an explicit reflection:

- What TRACK records as actually done, no-op, partial, blocked, or failed;
- how current observable state confirms the relevant result, rather than accepting an executor claim; and
- what verified discovery made the former future strategy invalid or incomplete.

Only TRACK discoveries marked verified may become task premises. An unconfirmed report needs its own validation task only when the capability required to perform that validation is observably available now; otherwise it stays out of future-plan facts. Verified unavailability of the exact capability required by a partial task does not justify another validation task: apply the partial branch below.

### Resolve a partial trigger before generic regeneration

For a triggering task recorded `partial`, inspect the exact observation still required by its conjunctive `Done when` and `Verify by`, then choose exactly one branch. A partial may never produce both branches, neither branch, a new `Root:`, or an unrelated replacement outcome.

**Executable remainder.** Use this branch only when every capability, person, environment, access path, and resource required to perform the missing observation is observably available now.

1. Create exactly one direct continuation with the next unused numeric ID.
2. Copy `Root:` exactly from the partial trigger and set `Continues:` to that trigger's task ID. Do not use `Reopens:`.
3. Preserve the unresolved outcome and its required observation; do not replace it with a different task or a generic validation placeholder.
4. Insert the continuation at the attempted predecessor's former relative position: after every surviving task that preceded the trigger and before the first surviving task that followed it. Repoint dependencies to this new ID where the unresolved outcome is still required.
5. Complete a material replan and increment the PLAN version exactly once.

**Unavailable verification capability.** Use this branch when observable state shows that a capability, person, environment, access path, or resource required for the missing observation is unavailable now and no valid continuation inside the unchanged SPEC and current episode can obtain it.

1. Create no continuation and no validation task for that already-observed unavailability.
2. Preserve the trigger's `Root:` and Root-derived blocker identity.
3. Apply `replan exhausted` to that Root and episode, using the unchanged PLAN version and preserving eligible independent Roots.

If availability itself is unconfirmed and can be checked with an available capability, a narrowly scoped validation task may establish that fact. It must not replace or detach the partial remainder. If the required checking capability is itself observably unavailable, exhaust the partial trigger's existing Root instead of creating a task that cannot be verified.

### Then regenerate only invalid future work

For a partial trigger, these generic steps apply only after the exclusive branch above has selected an executable remainder. Exhaustion skips directly to `Replan exhausted`.

1. Remove every ID already present in TRACK. No attempted or completed ID survives in actionable PLAN.
   An attempted task never survives in PLAN, even when its remainder still needs a new task.
2. Identify unattempted tasks that still hold and those invalidated by current verified state.
3. Preserve every valid survivor unchanged: same ID, outcome identity, task contract, lineage, purpose, and relative order.
4. Add only the smallest necessary future outcomes, grounded in verified facts rather than old placeholders.
5. Give every new outcome or remainder the next unused numeric `T<number>` after the highest numeric ID in PLAN or TRACK. Never reuse an attempted ID and never use suffix IDs.
6. Preserve `Root:` across the lineage. A remainder uses `Continues:` naming its immediate attempted predecessor. Only the first replacement after a valid exhausted-blocker reopening uses `Reopens:` naming the last exhausted attempt.
7. Repoint every dependency that named attempted work to the new remainder, or remove it only when TRACK evidence shows the exact prerequisite state needed by the dependent is already satisfied. A partial task may satisfy only that narrower prerequisite while its own unresolved clause remains unsatisfied. No dependency may point to any ID in TRACK.
8. Increment `Plan version` exactly once, keep exactly one `Plan version:` field, and set `Replanned because:` to the specific verified trigger.
9. Preserve complete coverage of every Must requirement and acceptance criterion through TRACK `done`/`no_op` evidence plus remaining PLAN tasks.
10. Run the PLAN quality gate before writing the result. Do not retain prior PLAN text as history.

If observation shows no material strategy change, do not perform these steps. Apply only checkpoint maintenance and keep the version unchanged.

### Replan exhausted

When no valid continuation remains inside the unchanged SPEC and the triggering Root's episode limits, do not fabricate another remainder or continue that episode. Exhaustion is scoped to that `Root:` and episode; it is not an instruction to discard unrelated future strategy.

Return only this exact internal receipt; never write it into TRACK:

```text
outcome: replan exhausted
trigger_task_id: <T-id>
plan_version: <N>
Root: <root task ID>
Blocker: BLK-<slug>-<root-task-id>
```

Do not rename, omit, supplement, or persist its fields.

The resulting PLAN remains future-only and keeps the current version; do not increment a version solely to declare exhaustion. Remove the attempted trigger and every same-Root continuation in that episode. Preserve each valid task under another Root in its existing relative order. If such a task depended on attempted work, remove that dependency only when verified TRACK state satisfies the exact prerequisite it needs; otherwise the task is not independent and must leave actionable PLAN rather than execute across the exhausted lineage. PLAN contains no continuation beyond the attempt limit, attempted ID, dependency to an ID in TRACK, or different outcome reusing the exhausted ID.

`replan exhausted` is terminal for the lineage. It is terminal for the run only when no eligible future task under another Root remains. The executor, not planning, appends the named checkpoint; the composite root decides whether the remaining PLAN routes to Execute or Report.

This return is not a new artifact, gate, canonical state, or TRACK field.

### Reopening after verified blocker resolution

Reopening starts a new attempt episode only after Execute verifies or receives explicit attestation that the stable blocker from the exhausted task is resolved.

1. Read the complete SPEC, current PLAN, complete TRACK, current observable state, exhausted trigger, and matching resolution evidence.
2. Preserve the exhausted task's `Root:` and stable blocker identity.
3. Allocate the next unused numeric task ID after the highest ID in PLAN or TRACK.
4. Use `Reopens:` on the first replacement, naming the exhausted attempt; do not also use `Continues:`.
5. Write a future-only PLAN with exactly one version field and increment the prior PLAN version exactly once.
6. Preserve valid unrelated survivors, their identity, relative order, coverage, and dependencies.
7. Return material replan complete. Execute validates the PLAN and appends the existing named `replan reopened (plan version N)` checkpoint associated with the exhausted trigger and matching blocker.

Reopening does not add a gate, canonical state, artifact, or required TRACK field.

### Contract invariant

> Replanning changes the strategy. It never changes the definition of success.

Do not weaken, drop, reinterpret, or silently bypass a SPEC acceptance criterion or Must requirement. If the only path requires a contract change, stop and tell the user which criterion is unreachable, what verified state makes it unreachable, and the options: change the contract, choose a different approach, or accept partial completion. The user alone can amend success.

## Plan quality gate

Run this before writing an initial plan, a materially replanned plan, or checkpoint-maintained PLAN. Fix every missing item; do not ship a caveat.

```markdown
- Grounded in an actual observation of current state: Pass / Missing
- Every acceptance criterion accounted for by TRACK done/no_op, a remaining task, or an exhausted lineage that preserves it as an unsatisfied terminal residual: Pass / Missing
- Every Must-priority requirement accounted for by TRACK done/no_op, a remaining task, or an exhausted lineage that preserves it as an unsatisfied terminal residual: Pass / Missing
- Every task lists Covers, or a task that transitively depends on it does; no task sits outside every requirement path: Pass / Missing
- Every ID in Covers exists in the SPEC: Pass / Missing
- Every task has one observable Done when: Pass / Missing
- Every task has a concrete Verify by: Pass / Missing
- No task describes HOW instead of WHAT: Pass / Missing
- Dependencies are ordered correctly and acyclic: Pass / Missing
- Exactly one Plan version field exists: Pass / Missing
- No ID already present in TRACK remains in PLAN: Pass / Missing
- No dependency points to an ID already present in TRACK: Pass / Missing
- PLAN contains only future work: Pass / Missing
- Every surviving ID preserves the same outcome identity, task contract, lineage, purpose, and relative order: Pass / Missing / N/A
- Every new outcome or remainder has the next unused numeric ID after the highest numeric ID in PLAN or TRACK: Pass / Missing / N/A
- Every remainder uses Continues for its immediate attempted predecessor, or Reopens only on the first replacement after valid reopening; they never coexist on one task: Pass / Missing / N/A
- Every partial trigger resolves exclusively to one direct continuation at its predecessor's former relative position or to same-Root exhaustion; capability unavailability never produces an unverifiable validation task: Pass / Missing / N/A
- Every task carries Root, and Root is unchanged across every task in one lineage: Pass / Missing
- No task remains under an exhausted Root or depends on its unsatisfied state without a valid reopening: Pass / Missing
- Status no-op appears only in an initial zero-task PLAN when TRACK has no task records: Pass / Missing / N/A
- Plan version is unchanged for maintenance and incremented exactly once for material replan: Pass / Missing / N/A
- Success criteria are unchanged from the SPEC (replan only): Pass / Missing / N/A

Verdict: Ready / Not ready
```

Coverage is always evaluated against TRACK plus remaining PLAN, never PLAN alone. A criterion stays covered after its completed task leaves PLAN.

## Anti-patterns

**Action lists.** “Open X, edit Y, run Z” is a trajectory, not an outcome plan.

**Unfalsifiable tasks.** “Improve error handling” has no observable postcondition, so execution cannot detect failure.

**Planning from the goal alone.** A coherent plan without an observation is still a guess.

**History in PLAN.** Completed tasks belong in TRACK; PLAN must remain an actionable future strategy.

**Reissuing attempted IDs.** Reusing a partial, blocked, or failed ID either repeats side effects or makes the executor skip necessary remainder work.

**Changing a surviving ID's outcome.** Reusing an unattempted ID for a materially different task hides a new result behind old identity. Preserve the original outcome or allocate a new ID.

**Broken dependencies.** Leaving a dependency aimed at an attempted task deadlocks execution; repoint it to the new remainder or remove it only when already-satisfied state permits.

**Checkpoint churn.** Incrementing a plan version when feedback did not invalidate future strategy obscures history and violates deterministic maintenance.

**Silent scope drift.** Dropping a hard requirement because it became difficult changes the contract. Escalate it.

**Padding.** Do not manufacture tasks to make a small goal look thorough; the reverse-coverage gate catches irrelevant work.
