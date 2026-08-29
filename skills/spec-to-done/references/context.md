# CURRENT resume projection

This is a conditional internal execution reference. Read it completely only for a possible fresh **Delegated** execution resume before the root loads full `TRACK.md`. Inline execution, planning, replanning, reporting, maintenance already in progress, and every terminal or repair path skip this reference and use the canonical artifacts.

Its single purpose is to reduce the historical TRACK text loaded into the root working context. It never changes workflow state or authority.

## Authority

Four artifacts remain canonical:

```text
SPEC.md   contract and success
PLAN.md   future work
TRACK.md  execution, evidence, gates, lineage, blockers, and side effects
REPORT.md terminal result
```

The additional files are disposable views:

```text
CURRENT.candidate.md  untrusted builder output
CURRENT.md            verifier-owned projection or retry-control state
```

Neither additional file is evidence. Neither can satisfy a task, requirement, acceptance criterion, gate, blocker resolution, permission, or terminal classification. Neither may repair a canonical artifact. Either may be replaced or deleted.

When artifacts disagree:

```text
SPEC wins about the contract.
PLAN wins about future work.
TRACK wins about execution and evidence.
CURRENT never wins. Ignore it and read the canonical artifacts.
```

Deleting CURRENT removes only the optimization. Continue from full SPEC, PLAN, and TRACK without repeating a recorded ID or side effect.

## Terms

**Fresh resume:** a new root context enters an existing Ready execution work item before dispatching any task in that context. A Safe projection is valid only in the fresh resume that just built and verified it. Every later fresh resume treats every existing Safe body as stale and rebuilds from canonical artifacts or uses full TRACK.

**Clean dispatch resume:** canonical inspection shows that the unchanged executor would proceed to ordinary task selection. It has no pending interruption reconciliation, repair, immediate replan, exhausted-blocker handling, reopening, escalation, terminal route, stale-report route, credential request, or authority request.

**Projection cycle:** one builder assignment followed by one different verifier assignment. At most one cycle may start in one fresh resume.

**Resume delta:** canonical TRACK records appended after a Safe projection was accepted in the current root context. It is not another artifact. The root uses the operational state loaded once from accepted CURRENT plus these newly observed records until a full-TRACK path is required.

## Entry gate

The root applies this order before reading full TRACK for ordinary execution:

1. Confirm the work has a Ready SPEC and a future PLAN by ordinary artifact routing.
2. Select the existing execution mode.
3. If mode is Inline, stop here and use full TRACK. Do not create, read as state, verify, or update CURRENT.
4. If this is not a fresh resume before dispatch, stop here and use the execution context already in force.
5. If the route is already known to require reconciliation, repair, replan, reopening, reporting, credentials, or authority, stop here and use full TRACK.
6. Read only the `Projection control` block of an existing CURRENT. Never use an old Safe body as current state.
7. If CURRENT exists but its control block is missing, malformed, or ambiguous, stop here and use full TRACK. Do not infer or repair derived control.
8. If control is `Disabled` and the user has not explicitly requested a retry, stop here and use full TRACK.
9. If a projection cycle already ran in this fresh resume, stop here and use its Safe result or the full-TRACK fallback. Never run a second cycle.
10. Start one isolated builder assignment. Mark the cycle as used in this root context.

Absence of CURRENT never blocks execution. If no Safe projection is produced, read full TRACK.

## Isolation contract

A Safe projection requires three different responsibilities:

| Responsibility | May read | May write | Must not do |
|---|---|---|---|
| Root | SPEC, PLAN, old CURRENT control; then Safe CURRENT or full TRACK | canonical artifacts only through their existing owners | read full TRACK merely to build or verify; build or judge the candidate |
| Snapshot builder | current SPEC, PLAN, full TRACK | `CURRENT.candidate.md` only | read old CURRENT/candidate as source; execute work; modify canonical artifacts |
| Snapshot verifier | current SPEC, PLAN, full TRACK, candidate, prior control supplied by root | CURRENT and candidate cleanup only | perform contract work; modify canonical artifacts; accept by plausibility |

Builder and verifier are separate assignments with separate contexts, even if they use the same model. The task performer cannot act as either. If the host cannot provide the two isolated assignments, return to full TRACK; correctness remains unchanged.

## Builder assignment

Give the builder only the canonical paths and this contract:

```markdown
## Role
You are the snapshot builder. Produce a derived operational view, not a decision.

## Inputs
- current Ready SPEC.md
- current future-only PLAN.md
- complete canonical TRACK.md

Do not read CURRENT.md or an older candidate as source.

## Preflight
Before writing, inspect canonical state. Return exactly one of:
- `candidate written` — the unchanged executor would proceed to ordinary task selection;
- `full TRACK route: <reason>` — reconciliation, replan, exhaustion, reopening, reporting, authority, credentials, or another non-dispatch route is current;
- `canonical issue: <exact invariant and evidence>` — current canonical artifacts are malformed or contradictory.

## Output
On `candidate written`, write only CURRENT.candidate.md. Preserve every applicable protected item. Use compact state, not narrative. Do not state a next task, next route, replan instruction, report instruction, or reopening instruction. Do not modify SPEC, PLAN, TRACK, or REPORT.
```

A `full TRACK route` is normal fallback, not a mismatch. A `canonical issue` invokes the existing repair or escalation path; it is never relabeled as projection failure and no task dispatches.

## Candidate content

The candidate starts with:

```text
# CURRENT candidate
Projection status: Candidate — untrusted until independent verification
Derived only from: SPEC.md + PLAN.md + TRACK.md
```

It then records compactly, without invented identifiers or routing conclusions:

### Contract state

- every Must requirement and acceptance criterion;
- current satisfaction state for each required ID;
- evidence class for every claimed satisfaction;
- applicable constraints, must-not-happen rules, and safety rules.

### Plan state

- current PLAN version;
- future task IDs and dependencies needed to interpret eligibility safely;
- `[verified]` discoveries currently used as PLAN premises.

### Execution state

- every attempted task ID and status;
- `Root`, `Continues`, and `Reopens` exactly as canonical;
- current attempt episode, counts, and limits;
- current effective gate and its associated task;
- active or exhausted blocker identity, affected root, state, and resolution condition;
- pending user actions;
- destructive, irreversible, or non-repeatable effects and explicit repeat prohibitions;
- active unresolved items, risks, and material deviations;
- corrections and supersessions that affect current behavior.

### Provenance state

- `[verified]`, `attested`, `unverified`, and `[reported, unconfirmed]` labels preserved exactly in meaning;
- no absent evidence converted into satisfaction;
- no unverified or reported fact promoted into a PLAN premise.

Protected content may be compressed in wording, but it may not be omitted, weakened, merged ambiguously, or stripped of its task/requirement/blocker association. If an item’s exact wording is needed to preserve a prohibition or resolution condition, retain it.

CURRENT contains state, never an authoritative conclusion such as:

```text
Next task
Next route
Invoke replanner
Report now
Reopen blocker
```

The root derives every such decision through the unchanged execution rules.

## Verifier assignment

Give a different verifier the candidate, canonical paths, the prior mismatch count from old CURRENT control, and this narrow contract:

```markdown
## Role
You are the snapshot verifier. You did not create the candidate. Attack its fidelity and size; do not solve the product task.

## Order
1. Validate current canonical invariants and clean-dispatch eligibility.
2. Enumerate canonical protected items by class.
3. Compare each item with the candidate for presence, meaning, association, and evidence class.
4. Check that the candidate contains no routing or completion decision.
5. Construct the exact publishable CURRENT with its control header and the candidate operational body unchanged.
6. Measure normalized UTF-8 bytes exactly for that final CURRENT text and canonical TRACK.
7. Return one narrow verdict with an external receipt and apply its file action.

## Verdicts
- `Safe`
- `Projection mismatch: <exact discrepancies>`
- `Protected state too large: <exact sizes and irreducible protected classes>`
- `Full TRACK route: <reason>`
- `Canonical issue: <exact invariant and evidence>`
- `Exact measurement unavailable: <reason>`
```

The verifier must derive discrepancies from canonical artifacts, not from confidence in the builder. It is not a second builder: it must not repair, complete, reorder, rewrite, or re-summarize candidate content. For Safe publication, remove only the required candidate preamble, prepend the fixed Safe control header, and copy the candidate operational body byte-for-byte. Any body change is a rejection, not verification.

### Canonical checks precede projection checks

At minimum, reject dispatch as a canonical issue when current state contains:

- a recorded task ID still dispatchable in PLAN;
- a dependency aimed at attempted work;
- contradictory current lineage or attempt counts;
- an unresolved execution-to-TRACK, TRACK-to-gate, or gate-to-PLAN window;
- a missing, global, or unusable current gate;
- an unverified discovery already promoted into PLAN;
- `replan exhausted` reopened without matching verified or attested blocker resolution;
- conflicting work identity or contract readiness.

Do not increment projection mismatch state for canonical issues.

### Exact size gate

Normalize each file by replacing every `CRLF` pair with `LF`; perform no other transformation. Count exact UTF-8 bytes. Safe requires:

```text
3 * normalized_CURRENT_bytes <= normalized_TRACK_bytes
```

Measure the final publishable CURRENT, not only the candidate body. Token count, line count, character count, and visual estimates do not qualify.

If exact measurement is unavailable, publish no Safe projection and use full TRACK. This is availability fallback, not a candidate mismatch.

When size exceeds the cap:

- If duplication, commentary, redundant headings, or other removable non-protected text causes the excess, verdict is `Projection mismatch`.
- If a faithful minimal rendering of protected state still cannot fit, verdict is `Protected state too large`; never truncate, weaken, or omit protected state.

No installed utility is required. Use an exact capability already supplied by the host; otherwise fail closed to full TRACK.

## Publication and control

`CURRENT.candidate.md` is untrusted at all times. A leftover candidate after interruption is deleted or rebuilt from canonical artifacts; it is never promoted automatically.

The verifier owns publication:

### Safe

Write CURRENT with this minimal header followed by the verified operational body:

```text
# CURRENT
Derived projection: not evidence; canonical artifacts always win
Projection control: Safe
Consecutive mismatches: 0
Scope: this fresh resume only
```

Construct the final text with that fixed header before measuring, so the size check has no self-referential field. If the exact gate passes, write it, delete the candidate, and return `Safe` with the exact CURRENT and TRACK sizes as the verifier receipt. The receipt is transient coordination, not evidence and not another artifact. A Safe result resets consecutive mismatches to zero.

### Projection mismatch

Do not preserve the rejected body. Increment the prior consecutive mismatch count and write control-only CURRENT:

```text
# CURRENT
Derived projection: not evidence; canonical artifacts always win
Projection control: Retryable | Disabled
Consecutive mismatches: 1 | 2
Reason: <exact candidate discrepancies>
Operational body: none; use full TRACK
```

Use `Retryable` for the first consecutive mismatch and `Disabled` for the second or later mismatch; store `2` as the saturated count for every second-or-later mismatch. Delete the candidate and use full TRACK. An explicit later retry permits one fresh-resume cycle; another mismatch remains Disabled, while Safe resets the count.

### Protected state too large

Write control-only CURRENT with `Projection control: Disabled`, the prior mismatch count unchanged, the exact sizes, and `Operational body: none; use full TRACK`. This is immediate disablement, not truncation and not a mismatch count. Explicit retry is required.

### Full-TRACK route, canonical issue, or unavailable measurement

Publish no Safe body. Delete any candidate. Preserve prior mismatch control when present, ignore every stale Safe body, and use full TRACK. A canonical issue follows existing repair/escalation before dispatch.

Projection control exists only in CURRENT. Never copy it into TRACK, PLAN, SPEC, or REPORT.

## Root use after Safe

For the fresh resume that just received Safe, the root reads once:

```text
full SPEC + full PLAN + verified CURRENT + verifier receipt
```

It does not load historical TRACK merely for ordinary task selection. It applies the existing dependency, attempted-ID, gate, and routing rules itself; CURRENT does not choose for it.

CURRENT is a one-time resume input, not a task-loop artifact. Once the first dispatch begins, do not read, create, verify, or update either projection inside the loop. The operational state already loaded from Safe CURRENT remains ordinary root working context and is extended only by canonical records observed and appended in this root context.

After each task, the root still:

1. verifies the postcondition independently;
2. appends the complete canonical TRACK record;
3. runs the unchanged replan gate;
4. applies same-version PLAN maintenance when the plan holds.

Do not update or reread CURRENT after a task. Keep newly appended records as the current root’s resume delta. The current PLAN and newest canonical delta override overlapping state loaded at resume. That loaded state plus the delta is only a working view; full TRACK remains the record.

Immediately stop using the projection as a TRACK substitute and load full TRACK before:

- reconciling any interrupted write window;
- repairing malformed canonical state;
- invoking material replanning for `replan required`;
- handling `replan exhausted` or `replan reopened`;
- producing or regenerating REPORT;
- escalating a contract, authority, credential, or destructive-action issue.

The replanner always receives `SPEC + PLAN + full TRACK + observable state`. The reporter always receives `SPEC + PLAN + full TRACK`. CURRENT never triggers, suppresses, delays, or supplies evidence to either.

## Literal preservation rules

This optimization does not modify any existing rule about:

- execution-to-TRACK recovery;
- TRACK-to-gate recovery;
- gate-to-PLAN comparison;
- higher PLAN version producing `replan done`;
- same PLAN version with `replan required` invoking the replanner;
- `replan exhausted` remaining terminal until matching verified or attested resolution;
- attempted IDs never being redispatched;
- one root plus at most two continuations per episode;
- stable `Root`, immediate `Continues`, episode-opening `Reopens`, and root-derived blocker identity;
- maintenance retaining PLAN version and material replanning incrementing it;
- verified-only discoveries becoming PLAN premises;
- SPEC remaining immutable to replanning.

If applying CURRENT would change any one of these results, the projection is unsafe: discard it and use full TRACK.

## Decision table

| Situation | Projection action | Workflow input |
|---|---|---|
| Inline mode | none | full TRACK |
| Eligible clean Delegated fresh resume | one builder/verifier cycle | Safe CURRENT or full TRACK |
| Builder/verifier isolation unavailable | none | full TRACK |
| Candidate mismatch | Retryable/Disabled control only | full TRACK |
| Protected state cannot fit | Disabled control only | full TRACK |
| Exact bytes unavailable | no Safe | full TRACK |
| Canonical violation | no Safe; repair/escalate | full TRACK |
| Task loop after Safe | do not read or refresh projections; append TRACK | loaded resume state + current resume delta |
| Replan, exhaustion, reopening, or report | no build/refresh | full TRACK |
| CURRENT absent or deleted | no authority lost | full TRACK unless a new eligible cycle independently succeeds |
