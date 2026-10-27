# Report

Produce the smallest developer-facing terminal report that preserves every material fact about the outcome. The report is the projection of **contract × final state × evidence**, never an executor's summary.

Two rules govern every decision:

> No material fact may be lost merely because the report is concise.
>
> No execution detail survives merely because it happened.

## Boundary

This procedure communicates only. It does not plan, replan, execute, independently verify, or repair.

Use it only when the run has reached a terminal path: every planned task is `done` or `no_op`; the plan is explicitly no-op; a blocker cannot be routed around; no valid continuation remains; the continuation bound is exhausted; or the user asked to stop. If valid work remains and the run is not terminal, return control to the composite root for the appropriate internal procedure. Do not fabricate terminality to make a report look finished.

Always produce exactly one of `COMPLETED`, `PARTIAL`, `BLOCKED`, `FAILED`, or `NO_OP` on every terminal path.

## Inputs

Read `spec-interview/<slug>/`:

- **Contract:** `SPEC.md`, always. Every Must-priority requirement and every acceptance criterion in it is a mandatory completion gate. `PLAN.md` and `TRACK.md` record how the contract was pursued; neither is ever the contract.
- **TRACK.md:** the whole record — the sealed segments under `track/` in order, then the active `TRACK.md` — holding the append-only history of task status, coverage, state delta, evidence, verification, unresolved work, risk, user action, deviation, assumptions, and checkpoints.
- **PLAN.md:** the goal statement, remaining tasks, no-op declaration, and criteria with no TRACK entry.

Without TRACK, reconstruct material facts only from observations that actually happened. Never reconstruct the requested goal from an execution trace and never invent evidence, history, or a success claim.

## 1. Classify before writing

First compare the final state to the contract and its evidence. Then select exactly one terminal classification using these mutually exclusive conditions, in order:

| Status | Select when |
|---|---|
| `NO_OP` | The requested state was already true and no change was needed: every relevant TRACK task is `no_op`, or PLAN declares `Status: no-op` with no tasks and supplies `Already true because` plus evidence. |
| `COMPLETED` | Every required criterion is satisfied by current evidence. |
| `BLOCKED` | A required outcome remains and external access, information, permission, or a user decision prevents valid continuation. |
| `FAILED` | A required outcome was not reached and no valid continuation is available, for reasons other than an external blocker. |
| `PARTIAL` | A meaningful subset is achieved but one or more required outcomes remain unsatisfied on an otherwise terminal path. |

`PARTIAL` is not a substitute for an active run: return control to the root if the remaining work can still proceed. `FAILED` is not a cleaner name for an unverified result. `BLOCKED` names the external dependency even when prior work is also partial.

Never upgrade `PARTIAL`, `BLOCKED`, `FAILED`, unresolved, or unverified work to `COMPLETED`. `attested` evidence can satisfy a criterion, but name it as a human confirmation rather than a measurement.

## 2. Compare final evidence to the contract

Build a private coverage table before compressing. It is scaffolding, not necessarily report output.

Join every TRACK `Covers:` ID to its required FR and AC in the SPEC, then to the evidence and the final satisfaction state. Judge each criterion across all entries, rather than trusting a task's final adjective.

Use SNAPSHOT coverage pointers to locate entries, including corrections and unresolved items in sealed segments; inspect the source evidence before classifying, following any external reference to its preserved version. A missing or changed evidence source is a verification gap, never proof of satisfaction. Missing or incomplete pointers require searching TRACK, never assuming satisfaction or absence of a gap.

```text
AC-001  satisfied     verified    integration test passed
AC-002  satisfied     attested    user confirmed the accepted invitations
AC-003  unsatisfied   —           test database unavailable
AC-004  unverifiable  unverified  production observation is unavailable
```

Apply these evidence rules:

- A criterion is satisfied only when final evidence is `verified` or `attested` and no material unresolved item still applies to it.
- `attested` means an explicit human confirmation. It satisfies the criterion but is reported as confirmed, not measured or tested.
- `unverified` is not satisfied evidence and cannot support a `COMPLETED` claim.
- When multiple entries cover one criterion, combine them. Later verified or attested evidence may close an earlier unverified gap only when it actually observes or confirms the previously missing outcome.
- An `Unresolved` item keeps its criterion unsatisfied until a later TRACK entry explicitly closes that item. A failed check, failed task, or destructive consequence remains material unless later final-state evidence explicitly supersedes it.
- If coverage or history is missing, report that limit; do not assume a criterion was satisfied.

The reporter may describe current verified facts but may not perform the verification that would establish them.

## 3. Select material facts and run the semantic checksum

Report consequences, not activity. A fact is material if omitting it could change the developer's understanding of the achieved outcome, changed behavior or interface, contract satisfaction, evidence strength, remaining risk, incomplete work, required user action, compatibility, or data safety.

```text
NON-MATERIAL                         MATERIAL
opened five files                    omit
ran a search                         omit
retried a command                    omit unless it changed final state
modified IUserRepository             the public repository interface now requires
                                     email lookup; other implementations must add it
```

For every material fact, assign exactly one disposition before rendering:

- `REPORT` — state it directly.
- `MERGE` — carry it fully in one other reported statement.
- `OMIT` — it is implied by a reported fact or has no user-visible consequence.

These facts may never be silently omitted when present:

- an achieved material outcome, changed external behavior, public contract, or interface;
- a changed user-data state, destructive action, or irreversible consequence;
- every unsatisfied criterion, failed verification, unverified completion claim, and missing-history limit;
- an unresolved blocker, remaining risk, material assumption, or required user action;
- a material deviation from the contract.

Read all relevant TRACK fields — `State delta`, `Evidence`, `Verification`, `Unresolved`, `Risk`, `User action`, `Deviation`, and material `Assumption` — before deciding the checksum. Filenames may follow a consequence for traceability; they never replace it. This checksum is a coverage check, not a word-count target.

For blocker guidance, join the task with applicable terminal/correction checkpoints and use the latest grounded guidance for the same task, episode, and blocker. Carry the impediment, affected outcome, consequence, missing condition, and resumption condition into the sections below; USER ACTION includes only required user action. Alternatives/recommendations need evidence. If guidance is missing, return to the root naming the specific gap for Evidence maintenance in Execute; do not investigate. A correction identifying the concrete unknown condition satisfies this request even when no solution is known; use it rather than requesting the same guidance again.

## 4. Distinguish state from evidence

Keep these states separate:

| State | Meaning in the report |
|---|---|
| implemented | A change or outcome exists, but may not be independently checked. |
| verified | A check, observed state, tool confirmation, or inspected artifact supports it. |
| attested | A person explicitly confirmed it; name the confirmation. |
| not verified | Evidence is absent, failed, unavailable, or insufficient. |

Every success claim must trace to a satisfied criterion, passed check, observed state, tool confirmation, inspected artifact, or explicit human confirmation.

```text
Wrong: The system is working correctly.
Right: The parser change is implemented. Unit tests pass; integration behavior was not verified.
```

When verification is unavailable, state the gap in one concrete clause. Do not hide it, apologize for it, or infer success from activity.

## 5. Render and persist one semantic result

Render only sections that contain material information, in this exact order:

1. `OUTCOME` — exactly one terminal classification and the contract-level result.
2. `MATERIAL CHANGE` — achieved behavior, interfaces, compatibility, data, destructive effects, deviations, and assumptions.
3. `VERIFICATION` — checks, evidence class, failures, and unverified limits.
4. `RESIDUAL STATE` — unsatisfied criteria, blockers, risk, and remaining gaps.
5. `USER ACTION` — only required action from the user.

Do not emit empty headings. A simple success may be two sentences without headings. Put cause and consequence in the same sentence; prefer concrete nouns and verbs, active voice when the actor matters, one stable term per concept, and one proposition per sentence when combining them would be ambiguous. Report state, not self: write `The build passes`, not `I successfully ran the build`. Avoid vague pronouns, decorative transitions, and synonyms chosen only for variety.

There is no hard word limit: hard limits cause omission. Apply **coverage first, compression second, style third**. Remove semantic duplicates, merge facts that share a consequence, and remove implementation details with no consequence; never add prose merely to make the result seem substantial.

Forbidden:

- execution narration with no consequence;
- unsupported success claims or generic success language;
- invented next steps;
- restating the request before its outcome;
- duplicating a fact; and
- hiding a failure, unverified state, destructive consequence, or residual risk.

New material TRACK information requires a new report body. With unchanged material state and adequate guidance, confirm and reuse the persisted body without writing or archiving. Immutability applies to each body; replacing the current projection is permitted only through Report preservation below.

For a new report, construct the concise body once. When a working directory exists, persist `spec-interview/<slug>/REPORT.md` with the current date and exactly one status in its header, followed by that body, using preservation below. The conversation receives that same body; the header is persistence metadata. Reuse skips construction and writes. Without a working directory, deliver only the conversation report.

Mechanical terminal handoff is mandatory:

1. Construct one body without further mutation, or reuse the adequate body already persisted.
2. For a new body, preserve the old REPORT and persist the header followed by the new body as below. Reuse confirms existing persistence and requires no rewrite.
3. Return to the root for finalization and emission of exactly the persisted body, byte-for-byte—without the file header or any prefix, suffix, summary, transcript, routing detail, or replacement validation narration. Evidence maintenance leaves all work statuses unchanged.

Never recreate the response from memory after writing `REPORT.md`. A paraphrase with equivalent meaning still violates the single-body output contract.

### Report preservation

Before replacing an existing REPORT, copy the whole file, header included, to this work's `reports/REPORT-<nnnnnn>.md`, numbered one above the highest existing archive (`000001` first). Read the copy back and compare it with the current REPORT; replace REPORT only after they match. If the copy cannot be completed, leave REPORT untouched and do not finalize. Use only ordinary file reads and writes; no hash, script, or other tool is required.

Archives are immutable, with two recovery cases after an interruption. If the highest archive already equals the current REPORT, preservation is done: write the new REPORT without archiving again. If it is a strict prefix of the current REPORT, it is an incomplete copy: complete it in place, then compare. Any other existing archive is never overwritten. Initial publication and unchanged-body reuse create no archive.

## Shapes to follow

**COMPLETED**

```text
OUTCOME
COMPLETED — Authenticated user lookup by email satisfies the required criteria.

MATERIAL CHANGE
The repository interface now requires email lookup, so other implementations must add it.

VERIFICATION
The build and 38 relevant tests pass.
```

**COMPLETED with attested evidence**

```text
OUTCOME
COMPLETED — All six candidate interviews are scheduled in accepted slots.

VERIFICATION
Each candidate confirmed the slot in writing; this is human confirmation, not a measured check.
```

**PARTIAL**

```text
OUTCOME
PARTIAL — The service implementation is complete, but the database-dependent criterion is unsatisfied.

VERIFICATION
The build passes. The integration test could not run because the test database is unavailable.

RESIDUAL STATE
Database-dependent behavior remains unverified.
```

**BLOCKED**

```text
OUTCOME
BLOCKED — Production deployment cannot continue without production credentials.

VERIFICATION
Version 2.4.0 passed staging checks; production was not changed.

USER ACTION
Provide production deployment access to continue.
```

**FAILED**

```text
OUTCOME
FAILED — The required migration state was not reached and no valid continuation remains.

VERIFICATION
The migration verification failed because the target schema rejects the required constraint.

RESIDUAL STATE
Existing user data was not changed.
```

**NO_OP**

```text
OUTCOME
NO_OP — No change was needed; nullable reference types are already enabled globally.

VERIFICATION
Project configuration inspection confirms the setting.
```
