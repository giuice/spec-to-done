---
name: spec-to-done
description: Use for substantial create, build, implement, launch, redesign, migrate, or end-to-end outcomes and to resume their SPEC-to-REPORT work. For an explicitly requested single stage, satisfy its prerequisites and stop at that stage. Do not use for a trivial reversible one-outcome change with no product, migration, data, security, integration, or new failure behavior; when unsure, use this skill.
---

# Spec to Done

## Composite ownership

Own **SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT** using only this tree. Never invoke another skill or ask the user to invoke a stage. Name the selected local reference, read it completely, apply it, return here, inspect again, and continue automatically. Pause only for a real interview answer, smallest ambiguity decision, authority, credentials, or external state.

## Artifact-first lookup and boundary

Derive a kebab-case slug and inspect `spec-interview/<slug>/` first. SPEC, state, rounds, PLAN, TRACK (with its sealed `track/` segments), SNAPSHOT, and REPORT belong only under that directory—never at the workspace root. Product deliverables stay at their contract paths. Match stored Goal, SPEC summary, or interview restatement to the actual request, not just the slug. A different goal receives a distinct folder; uncertain identity needs the smallest decision. Preserve malformed, premature, partial, and conflicting evidence—never delete, overwrite, or silently repair it.
Each work's `state.md` opens with `Status: active | frozen | closed`: at most one `active` repository-wide, `frozen` preserved as documentation except for the evidence maintenance below, `closed` terminal after its report.
The root owns this line. Starting new work, explicitly resuming a `frozen` or `closed` work whose next stage is Specify or Plan, or entering an authorized, permitted execution continuation makes it `active` and freezes the previously active work, stated rather than assumed. Selecting or naming existing work for inspection, or replying to its REPORT, does neither. Never restore another work implicitly.

An unequivocal reply concerning a recorded impediment or required pending action identifies the REPORT's work from conversation context or the message link, even when closed and without a slug; never choose simply the newest REPORT on disk. Ambiguity requires the smallest identity decision before changing any work. A report's classification or `USER ACTION` heading alone is insufficient: an optional action after completion does not resume execution. Selection allows evidence maintenance without changing any `state.md`; for work whose next stage is Execute or Report, only Execute's established valid continuation permits activation.

Every work item governed by this skill requires a Ready SPEC approved by the user. Specify records the user's confirmation of the current understanding in state.md before declaring readiness; approval does not replace the readiness gate. There is no no-SPEC execution mode. A stage-only request changes the stopping boundary, not the required prerequisites: run every missing predecessor through the requested stage, then stop before the next one, so a plan-only request without a Ready SPEC is `Specify → Plan → stop`. Any substantial new work begins with Specify, including a detailed brief or handoff. A supplied SPEC, PRD, plan, or prior context is interview input, not a contract: Specify is bypassed only when `spec-interview/<slug>/state.md` already records per-domain coverage and `Verdict: Ready`, which only Specify writes. Use supplied documents or prior context as interview answers only when the user explicitly asks or authorizes it; otherwise use them to inform questions. Interview the gaps and, if none remain, ask for additions or corrections and wait for the user's confirmation before declaring readiness. Trivial means reversible + one outcome + no product, migration, data, security, integration, or new failure behavior; doubt specifies. For unclear non-product discussion with no checkable outcome, clarify only done, must-not-happen, and constraints; do not force a product interview.

## Artifact-state routing

Use ordinary inspection of artifacts, timestamps, contents, statuses, and checkpoints. First applicable row wins; consume an internal stage return before reconsidering the same user message. Plan and execution routes require a Ready SPEC approved by the user; missing readiness or approval returns to Specify with all evidence preserved.

| State | Declared outcome |
|---|---|
| Reply to a REPORT's user action or blocker, but work identity is ambiguous | Ask the smallest identity decision before activating or freezing any work. |
| Execute establishes an authorized valid continuation | Activate the selected work, then return to Execute for its existing reconciliation/reopening procedure. Selection or new evidence alone is insufficient. |
| Execute returns maintenance with no valid continuation | Read `references/report.md` completely; preserve/replace or reuse the report, then finalize maintenance without re-examining the same reply. |
| `closed` or `frozen` work identified by a reply about its recorded impediment/required pending action or by an explicit execution-resume request, or a reporter request naming missing guidance; `active` work uses its ordinary rows below | Name and read `references/execute.md` completely; use Evidence maintenance under Resuming before ordinary execution recovery. The root neither verifies resolution nor writes TRACK. Ready SPEC and approval remain prerequisites. |
| `state.md` Status is `frozen` or `closed`, without a permitted continuation or maintenance route above or an explicit Specify/Plan-stage resumption activated as above | Documentation only: do not advance execution or reuse its slug. |
| Empty substantial work | Read `references/specify.md` completely; specify. |
| Interview rounds/state without a Ready SPEC | Read `references/specify.md` completely; preserve and resume. |
| Supplied SPEC/PRD/plan/context without this workflow's `state.md` readiness record | Read `references/specify.md` completely; treat it as interview input and specify. Never route it to Plan on its own strength. |
| Ready SPEC approved by the user only | Read `references/plan.md` completely; plan. |
| Terminal state, including an effective `replan exhausted` gate without confirmed resolution | Read `references/report.md` completely; preserve the gate. Reuse an adequate current body or persist its replacement. |
| Ready SPEC approved by the user + future PLAN, with or without TRACK | Read `references/execute.md` completely; execute, checkpoint, or replan as state requires. |
| Current REPORT and unchanged earlier artifacts, without a maintenance/continuation route above | Reuse the persisted body; do not redo work or change work statuses. |
| REPORT stale because an earlier artifact or implementation evidence changed | Preserve stale REPORT; select the next active stage from current artifacts. |
| PLAN/TRACK/implementation without a Ready SPEC | Preserve all evidence; read `references/specify.md` completely. Replan waits for Ready. |
| Malformed or contradictory artifacts, multiple goals, uncertain identity | Preserve evidence and ask the smallest resolving decision. |

A detailed handoff is not a Ready SPEC. Completion follows through the reporter, never a silent stop.

## Continuation and safety

After every selected reference return here and route again. Execute reads the local plan reference for replanning and the local report reference for every terminal route; both are internal continuation, not cross-skill handoff. TRACK — its sealed segments plus the active file — is the sole execution record and its task records, corrections, and gate checkpoints are append-only; `SNAPSHOT.md` is a derived current-state view rewritten at every gate, and TRACK wins any disagreement. Before a destructive or irreversible action, preserve evidence, ask for authority, and report the blocker rather than proceeding.

On a terminal route, treat the report body as a single immutable output: persist it after the `REPORT.md` header or confirm the reused body is already persisted, then emit it byte-for-byte. Do not reconstruct, paraphrase, prefix, suffix, or replace it with routing, execution, or validation narration. Emission ends this invocation; a persistence failure pauses it until the condition changes.

Terminal finalization has one order: reporting persists the replacement or confirms the adequate body already persisted; the root closes an execution invocation's `state.md`; then the root reads and emits exactly that body. Evidence maintenance and unchanged-report reuse leave every work's status untouched, including the selected work. Execute never persists REPORT or emits the final response. A run whose PLAN still contains actionable work has not reached ordinary completion; `closed` records that an execution invocation ended, never that the contract was satisfied. A reporter's specific guidance gap routes to evidence maintenance, not back to reporting without progress; a recorded concrete unknown condition is sufficient guidance when no solution is established.

## Full protocol
A lineage permits one root attempt and at most two continuation attempts per episode, for a maximum of three attempts: T1 is the root, T2 and T3 are continuations, and a fourth attempt in the same episode is forbidden. Use a stable `Blocker: BLK-<slug>-<root-task-id>`. Reopen only a matching exhausted blocker after verified or explicitly attested resolution, append evidence and a new episode checkpoint, and preserve history. Reconcile execution→TRACK, TRACK→gate, and gate→PLAN before future work; never repeat a recorded ID or completed side effect.
