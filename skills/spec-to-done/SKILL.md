---
name: spec-to-done
description: Use for substantial create, build, implement, launch, redesign, migrate, or end-to-end outcomes and to resume their SPEC-to-REPORT work. Do not use for an explicitly requested single stage or a trivial reversible one-outcome change with no product, migration, data, security, integration, or new failure behavior; when unsure, use this skill.
---

# Spec to Done

## Composite ownership

Own **SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT** using only this tree. Never invoke another skill or ask the user to invoke a stage. Name the selected local reference, read it completely, apply it, return here, inspect again, and continue automatically. Pause only for a real interview answer, smallest ambiguity decision, authority, credentials, or external state.

## Artifact-first lookup and boundary

Derive a kebab-case slug and inspect `spec-interview/<slug>/` first. SPEC, state, rounds, PLAN, TRACK, and REPORT belong only under that directory—never at the workspace root. Product deliverables stay at their contract paths. Match stored Goal, SPEC summary, or interview restatement to the actual request, not just the slug. A different goal receives a distinct folder; uncertain identity needs the smallest decision. Preserve malformed, premature, partial, and conflicting evidence—never delete, overwrite, or silently repair it.
Each work's `state.md` opens with `Status: active | frozen | closed`: at most one `active` repository-wide, `frozen` preserved as documentation only, `closed` terminal after its report.
The root writes this line on entry and updates it on terminal exit; naming a `frozen` or `closed` work makes it `active` and freezes the previously active one, stated rather than assumed.

Every work item governed by this skill requires a Ready SPEC. There is no no-SPEC execution mode. A stage-only request changes the stopping boundary, not the required prerequisites: run every missing predecessor through the requested stage, then stop before the next one, so a plan-only request without a Ready SPEC is `Specify → Plan → stop`. Any substantial new work begins with Specify, including a detailed brief or handoff. A supplied SPEC, PRD, plan, or prior context is interview input, not a contract: Specify is bypassed only when `spec-interview/<slug>/state.md` already records per-domain coverage and `Verdict: Ready`, which only Specify writes. Otherwise extract what it already answers and interview the gaps. Trivial means reversible + one outcome + no product, migration, data, security, integration, or new failure behavior; doubt specifies. For unclear non-product discussion with no checkable outcome, clarify only done, must-not-happen, and constraints; do not force a product interview.

## Artifact-state routing

Use ordinary inspection of artifacts, timestamps, contents, statuses, and checkpoints. First applicable row wins.

| State | Declared outcome |
|---|---|
| `state.md` Status is `frozen` or `closed`, and the request does not name that work | Documentation only: do not route into it, advance it, or reuse its slug. |
| Empty substantial work | Read `references/specify.md` completely; specify. |
| Interview rounds/state without a Ready SPEC | Read `references/specify.md` completely; preserve and resume. |
| Supplied SPEC/PRD/plan/context without this workflow's `state.md` readiness record | Read `references/specify.md` completely; treat it as interview input and specify. Never route it to Plan on its own strength. |
| Ready SPEC only | Read `references/plan.md` completely; plan. |
| SPEC + future PLAN, with or without TRACK | Read `references/execute.md` completely; execute, checkpoint, or replan as state requires. |
| No remaining task / terminal execution state | Read `references/report.md` completely; report. |
| Current REPORT and unchanged earlier artifacts | Terminal/current: state outcome, do not redo work. |
| REPORT stale because an earlier artifact or implementation evidence changed | Preserve stale REPORT; select the next active stage from current artifacts. |
| PLAN/TRACK/implementation without a Ready SPEC | Preserve all evidence; read `references/specify.md` completely. Replan waits for Ready. |
| Malformed or contradictory artifacts, multiple goals, uncertain identity | Preserve evidence and ask the smallest resolving decision. |
| Exhausted blocker | Read `references/execute.md` completely only after a matching blocker resolution is verified or explicitly attested; otherwise read `references/report.md` completely. |

A detailed handoff is not a Ready SPEC. Completion follows through the reporter, never a silent stop.

## Continuation and safety

After every selected reference return here and route again. Execute reads the local plan reference for replanning and the local report reference for every terminal route; both are internal continuation, not cross-skill handoff. TRACK is the sole execution record and its task records, corrections, and gate checkpoints are append-only. Each record is written compact at the point of writing — identifiers and protected state literal, state and evidence reduced to their material form, narrative and raw logs never written — and nothing already recorded is ever summarized. Before a destructive or irreversible action, preserve evidence, ask for authority, and report the blocker rather than proceeding.

On a terminal route, treat the report body as a single immutable output: persist it after the `REPORT.md` header, then emit that exact body byte-for-byte as the developer response. Do not reconstruct, paraphrase, prefix, suffix, or replace it with routing, execution, or validation narration.

Terminal finalization has one order: execution reconciliation passes; reporting persists `REPORT.md`; the composite root changes `state.md` to `Status: closed`; then the root reads the persisted report body from disk and emits those exact characters without adding blank lines or Markdown formatting. A task record, an effective `replan required`, or future work still present in PLAN cannot enter this sequence.

## Full protocol
A lineage permits one root attempt and at most two continuation attempts per episode, for a maximum of three attempts: T1 is the root, T2 and T3 are continuations, and a fourth attempt in the same episode is forbidden. Use a stable `Blocker: BLK-<slug>-<root-task-id>`. Reopen only a matching exhausted blocker after verified or explicitly attested resolution, append evidence and a new episode checkpoint, and preserve history. Reconcile execution→TRACK, TRACK→gate, and gate→PLAN before future work; never repeat a recorded ID or completed side effect.
