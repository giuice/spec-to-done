# Plan From Specification

Produce a high-level, outcome-shaped plan that an executor can carry out, and keep its future strategy true as reality diverges. Planning owns decomposition, ordering, observable postconditions, and strategy repair. It does not execute work, decide what success means, write TRACK, or report the terminal outcome.

The boundary is absolute: **replanning may change how the goal is reached; it never changes what counts as success.**

## Artifacts and inputs

One work item lives in `spec-interview/<slug>/`:

```text
SPEC.md       the contract; mandatory and read-only to planning
PLAN.md       the actionable future strategy; written or maintained here
TRACK.md      append-only execution history; input only during a replan
REPORT.md     terminal communication; owned by reporting
```

Planning is self-contained. Use only the contract, current PLAN, TRACK, and observable current state available to the run. Do not depend on repository source snapshots, external documentation, scripts, provider-specific APIs, or another runtime skill.

## The SPEC is required

Planning operates on a Ready SPEC and nothing else. `spec-interview/<slug>/state.md` must record per-domain coverage and `Verdict: Ready`, which only Specify writes.

If no Ready SPEC exists, do not plan. Preserve every artifact already present, return control to the composite root for Specify, and let it route back here once readiness is recorded. A stated goal, a supplied PRD, a detailed handoff, or an existing PLAN is interview input, never a contract. Never invent a SPEC and never convert an existing PLAN into one.

Every plan therefore names its contract as `Spec: ./SPEC.md`, and every task carries `Covers:`.

## 1. Ground the plan in observation, never assumption

Before writing or regenerating tasks, inspect actual current state. Use what the domain makes observable: files and structure, configuration, running behavior, available tools, data, prior work, external services, or people.

The reasoning for `T1` must cite a concrete observation: what exists, what is missing, and which contract assumptions it confirms or contradicts. Do not infer a premise merely because a goal mentions it.

If observation contradicts the contract, stop and escalate the contradiction. Do not quietly plan around it. If the goal is already true, produce no work merely to make a plan look substantial. Write this zero-task plan and return control to the composite root for reporting:

```markdown
# PLAN: <slug>

Spec: ./SPEC.md
Goal: <one sentence>
Plan version: 1
Status: no-op
Already true because: <the observation that establishes it>
Evidence: <how it was observed>
```

`Status: no-op` appears only on a zero-task, already-satisfied plan. A PLAN with tasks omits it.

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
Depends on: —

## T2 — <outcome-shaped title>
Reasoning: <why this future outcome is needed>
Task: <what must become true>
Done when: <observable postcondition>
Verify by: <evidence-producing check>
Covers: FR-002
Depends on: T1
```

Field rules:

- IDs are stable and never reused. A surviving task retains its ID across plan versions; a new task takes the next unused number from PLAN and TRACK. IDs attached to completed or attempted work are historical and cannot be recycled.
- `Covers:` is mandatory on every task and maps to requirement and acceptance-criterion IDs that exist in the SPEC. Every Must-priority requirement and every acceptance criterion must be covered by TRACK `done`/`no_op` entries plus remaining PLAN tasks.
- `Continues:` appears only on a replan task that plans the remainder of one `partial`, `blocked`, or `failed` task. It names that attempted ID. On a confirmed reopening after `replan exhausted`, the first replacement instead uses `Reopens:` to name the prior attempt and omits `Continues:`; this preserves history while starting a new attempt episode.
- `Depends on:` lists prerequisite task IDs; use `—` when none. Place every prerequisite before its dependent and keep the dependency graph acyclic.
- `Plan version` labels the strategy used for TRACK entries. PLAN is not an archive: it contains only actionable future work. TRACK preserves completed and attempted history.

## Initial planning

Use initial planning when a contract or checkable goal exists and no actionable PLAN exists.

1. Read the SPEC completely.
2. Observe current state and stop for a contradiction or write the zero-task no-op plan if it is already true.
3. Decompose only the remaining work into outcome-sized, checkable tasks.
4. Map all Must requirements and acceptance criteria to TRACK history plus future tasks.
5. Run the quality gate below, fix every failure, and write version 1.

Planning stops at the written strategy and returns control to the composite root. It never performs planned work.

## Checkpoint maintenance without a strategy replan

Every execution checkpoint distinguishes maintenance from replanning. When verified feedback leaves the remaining strategy valid, do not regenerate it or increment `Plan version`. The executor records the completed or no-op task in TRACK; maintenance removes that now-completed task from actionable PLAN, leaves surviving task IDs and dependencies unchanged, and preserves the same plan version. Its coverage remains satisfied through TRACK.

This deterministic maintenance is required even when no future work changes: completed work must not remain in PLAN, and an unchanged feedback checkpoint is not a pretext for needless strategy churn.

## Replanning only after material divergence

Enter replan mode only from the execution checkpoint when verified feedback makes future strategy false or incomplete, or when a `replan exhausted` run is reopened after the executor verifies or obtains user attestation that its blocker is resolved. Merely reaching the end of a successful task is not a replan trigger.

Inputs are the immutable SPEC, the current PLAN, the append-only TRACK, and current observed state.

### Reflect first

Start the first future task’s reasoning with an explicit reflection:

- What TRACK records as actually done, no-op, partial, blocked, or failed;
- how current observable state confirms the relevant result, rather than accepting an executor claim; and
- what verified discovery made the former future strategy invalid or incomplete.

Only TRACK discoveries marked verified may become task premises. An unconfirmed report needs its own validation task or stays out of future-plan facts.

### Then regenerate only invalid future work

1. Identify remaining tasks that still hold and those invalidated by current verified state.
2. Remove work that is no longer needed; retain valid surviving tasks unchanged, including their IDs.
3. Add the smallest necessary future outcomes, using concrete discovered facts rather than old placeholders.
4. Replace every attempted instruction with a new ID for its remainder. An attempted task never survives in PLAN, even if some of its work remains.
5. Repoint each surviving dependency that named a replaced task to the new task, or drop it only when the already-completed portion satisfies the prerequisite. No dependency may name an attempted (`partial`, `blocked`, or `failed`) TRACK task because it can never become a satisfiable prerequisite.
6. Increment `Plan version` and set `Replanned because:` to the specific verified trigger. Do not retain prior plan text as history.

If observation shows no material strategy change, leave strategy untouched and apply only checkpoint maintenance described above.

### Contract invariant

> Replanning changes the strategy. It never changes the definition of success.

Do not weaken, drop, reinterpret, or silently bypass a SPEC acceptance criterion or Must requirement. If the only path requires a contract change, stop and tell the user which criterion is unreachable, what verified state makes it unreachable, and the options: change the contract, choose a different approach, or accept partial completion. The user alone can amend success.

## Plan quality gate

Run this before writing an initial plan, a materially replanned plan, or checkpoint-maintained PLAN. Fix every missing item; do not ship a caveat.

```markdown
- Grounded in an actual observation of current state: Pass / Missing
- Every acceptance criterion covered by TRACK done/no_op or a remaining task: Pass / Missing
- Every Must-priority requirement covered by TRACK done/no_op or a remaining task: Pass / Missing
- Every task lists Covers, or a task that transitively depends on it does; no task sits outside every requirement path: Pass / Missing
- Every ID in Covers exists in the SPEC: Pass / Missing
- Every task has one observable Done when: Pass / Missing
- Every task has a concrete Verify by: Pass / Missing
- No task describes HOW instead of WHAT: Pass / Missing
- Dependencies are ordered correctly and acyclic: Pass / Missing
- PLAN contains only future work; completed tasks exist only in TRACK: Pass / Missing
- No attempted TRACK task is reissued or retained; each remainder uses a new ID with Continues, or Reopens on a confirmed reopening: Pass / Missing / N/A
- Surviving IDs remain stable and dependencies are repointed away from attempted tasks: Pass / Missing / N/A
- Success criteria are unchanged from the SPEC (replan only): Pass / Missing / N/A
- No strategy version changed for unchanged feedback; only completed work was removed (maintenance only): Pass / Missing / N/A

Verdict: Ready / Not ready
```

Coverage is always evaluated against TRACK plus remaining PLAN, never PLAN alone. A criterion stays covered after its completed task leaves PLAN.

## Anti-patterns

**Action lists.** “Open X, edit Y, run Z” is a trajectory, not an outcome plan.

**Unfalsifiable tasks.** “Improve error handling” has no observable postcondition, so execution cannot detect failure.

**Planning from the goal alone.** A coherent plan without an observation is still a guess.

**History in PLAN.** Completed tasks belong in TRACK; PLAN must remain an actionable future strategy.

**Reissuing attempted IDs.** Reusing a partial, blocked, or failed ID either repeats side effects or makes the executor skip necessary remainder work.

**Broken dependencies.** Leaving a dependency aimed at an attempted task deadlocks execution; repoint it to the new remainder or remove it only when already-satisfied state permits.

**Checkpoint churn.** Incrementing a plan version when feedback did not invalidate future strategy obscures history and violates deterministic maintenance.

**Silent scope drift.** Dropping a hard requirement because it became difficult changes the contract. Escalate it.

**Padding.** Do not manufacture tasks to make a small goal look thorough; the reverse-coverage gate catches irrelevant work.
