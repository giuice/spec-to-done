<!-- Pinned provenance: giuice/giuice-agent-skills, skills/spec-from-scratch; upstream snapshot commit 5efe8276889dba9ea06d522516a2396aabb626ec; last source-file change d396316c1d136e2a6fc0c9a9ccb1f143814539a9. -->

# Specify

Turn an unclear product idea into a complete, assumption-light Product SPEC. This is an internal stage of the composite: after a valid SPEC is persisted, return control to the composite root so it can continue. Do not draft a final SPEC until the user's goals, requirements, constraints, edge cases, business rules, and acceptance criteria are fully understood.

## Operating principle

The core job is not writing first; it is discovering what must be true before a useful SPEC can exist. A polished SPEC built on hidden assumptions is worse than no SPEC because it makes uncertainty look settled. Never answer an interview question for the user: extract facts they already supplied, distinguish them from unresolved decisions, and ask about the latter.

Every substantial new run uses this stage. The sole bypass is procedural, not qualitative: `spec-interview/<slug>/state.md` already records per-domain coverage and `Verdict: Ready`, which only this stage writes. A supplied document, detailed handoff, filename, prior conversation, or asserted provenance is not proof that a contract is complete.

## Resume, supplied contract, or new interview

Derive a short kebab-case slug and use `spec-interview/<slug>/`. If its `state.md` or prior round files exist, read them first, preserve answered material, and ask only the remaining or newly unlocked questions. If an existing folder belongs to a different goal, choose a distinct slug or ask the user before touching it.

A supplied `SPEC`, `PRD`, plan, or prior context is interview input, not a contract. So is a `PLAN.md`, a `TRACK.md`, or an existing implementation produced before the SPEC became mandatory: preserve all of it, read it as evidence of what was already attempted and what is already true, and never convert it into a SPEC. Once a Ready SPEC exists, planning starts from it and from observed state; attempted task IDs are not reused and recorded side effects are not repeated. Extract everything it already answers, record each extracted answer beside the passage it came from in `state.md`, and ask only the missing or ambiguous questions: a thorough document earns a short interview, never no interview. Do not persist it as `SPEC.md`, and do not plan or implement on its strength. However complete it looks, it cannot bypass this stage, because the bypass is satisfied by this stage's recorded coverage, not by a document.

Otherwise, restate the raw idea in one or two sentences, create the working directory, detect whether the project produces code, and begin round 1.

## Interview mode: exhaustive rounds

There is one mode: **exhaustive**. Each round asks the maximum set of useful questions at once, never one question per message. Depth comes from a coverage gate that forces follow-up rounds until every applicable domain is `Clear`.

- **Round 1** asks every question that does not depend on a prior answer, covering all domains at once.
- **Later rounds** add questions only when a prior answer unlocks them or the coverage gate marks a domain `Partial` or `Missing`. Defer dependent questions until the round after their prerequisite is answered.
- After each round, store the answers and update the coverage table before generating the next round.

Round 1 includes at least one question about goals, primary users, scope boundaries, business rules or constraints, edge cases/failure modes, and acceptance criteria/success signals. Aim wide--normally 7-10 or more questions--so the shape of the problem is exposed in one pass. Use answer options that teach tradeoffs. Mark exactly one option `(Recommended)` unless there is no sensible default. Every question must also allow a written answer or note; never force a fixed choice.

## 1. Deliver questions through a clickable UI

Use a host-provided structured question UI when it can carry the entire round. Always include an `Other - write your own` choice and tell the user that they may add a free-text note to any answer. If the host's question-count cap is below what the exhaustive round requires, do not shrink the round to fit it: use the HTML fallback for the full round. Native UI is appropriate for small follow-up rounds that fit.

When native UI is unavailable or cannot carry the full round, copy `assets/interview-round.template.html` to `spec-interview/<slug>/round-N.html`, set its round and topic values, and replace only its `QUESTIONS` array with the round's real questions. Keep its rendering and answer-return logic intact. The generated file must remain a single self-contained file with inline CSS and JavaScript and no network access. It renders radio buttons for single choice, checkboxes for multiple choice, a labeled `Other / note` textarea for every question, visible focus styles, a structured **Copy answers** route, and a **Download answers.json** fallback.

Tell the user to fill the form, click **Copy answers**, and paste the returned block into the chat (or share the downloaded file). When the block arrives, briefly confirm the parse by restating what each answer was read as before continuing. If answers are absent, malformed, contradictory, or insufficient to clear required domains, persist the state and ask for the smallest needed clarification; never fill in the missing decision.

## 2. Interview exhaustively by domain

Cover these domains before drafting:

1. **Problem and goals**
   - Problem being solved
   - Desired outcome
   - Primary and secondary users
   - Jobs to be done
   - Success metrics
   - Non-goals

2. **Scope and deliverables**
   - Product surface or system boundary
   - Must-have vs should-have vs later
   - User journeys
   - Inputs and outputs
   - Environments, platforms, or channels
   - Dependencies

3. **Requirements**
   - Functional requirements
   - Data requirements
   - Permission and access rules
   - Integrations
   - Reporting, notifications, or automation
   - Admin or support needs

4. **Business rules**
   - Eligibility rules
   - Pricing, billing, quotas, or limits
   - Workflow states and transitions
   - Approval rules
   - Compliance or policy constraints
   - Exceptions and overrides

5. **Constraints**
   - Technical constraints
   - Time, budget, staffing, and operational constraints
   - Existing systems that must be preserved
   - Security, privacy, legal, and regulatory constraints
   - Performance and reliability expectations
   - Localization, accessibility, and device constraints

6. **Edge cases and failure modes**
   - Empty, invalid, duplicate, stale, missing, or conflicting data
   - Partial completion and retry behavior
   - Offline or degraded dependencies
   - Abuse cases
   - Race conditions or concurrent edits
   - User mistakes and recovery paths

7. **Acceptance criteria and validation**
   - Observable behavior for each major flow
   - Given/When/Then scenarios
   - Definition of done
   - Test data or examples
   - UAT checklist
   - Launch or rollout criteria

8. **Test strategy** *(only when the project produces code)*
   - Target test levels: unit, integration, and end-to-end
   - TDD or test-after, and whether tests are written from acceptance criteria
   - Test data, fixtures, and mocks
   - Test environments and CI expectations
   - Coverage or done criteria for tests
   - Error and failure cases that must have tests

Skip the test-strategy domain entirely for non-code deliverables.

## 3. Track knowns and unknowns

After each round, persist answers to `spec-interview/<slug>/state.md` so progress survives a dropped chat. Rewrite the whole file each round.

**Every rewrite preserves `Status:` as the first line, byte for byte.** The composite root owns that line and is the only writer of its value; this stage carries it forward unchanged and never removes, rewrites, or silently repairs it. A rewrite that drops it destroys the repository-wide record of which work is active. The scope restatement from the first step follows it, so a later session can identify the work from the file alone, and then come the answers grouped by domain and this current-state view:

```text
Status: active | frozen | closed

<scope restatement>
```

```markdown
## Current understanding

| Area | Status | Notes |
|---|---|---|
| Goals | Clear / Partial / Missing | ... |
| Users | Clear / Partial / Missing | ... |
| Requirements | Clear / Partial / Missing | ... |
| Constraints | Clear / Partial / Missing | ... |
| Edge cases | Clear / Partial / Missing | ... |
| Business rules | Clear / Partial / Missing | ... |
| Acceptance criteria | Clear / Partial / Missing | ... |
| Test strategy (code only) | Clear / Partial / Missing / N/A | ... |

## Remaining unknowns

- [Unknown]
```

Do not mark a domain `Clear` on generic answers alone. Business rules and edge cases require concrete examples. While any required domain is below `Clear`, generate another round focused only on weak or newly unlocked areas; do not move on.

## 4. Apply the hard readiness gate

Before writing or accepting `SPEC.md`, explicitly record this check in `state.md`:

```markdown
## SPEC readiness check

- Goals are specific and measurable: Pass / Missing
- Users and stakeholders are identified: Pass / Missing
- In-scope and out-of-scope boundaries are explicit: Pass / Missing
- Functional requirements are testable: Pass / Missing
- Business rules are explicit: Pass / Missing
- Constraints are explicit: Pass / Missing
- Edge cases and failure modes are covered: Pass / Missing
- Acceptance criteria are observable: Pass / Missing
- Every Must-priority functional requirement has at least one acceptance criterion: Pass / Missing
- Every mandatory business rule, hard constraint, and must-not-happen condition is validated by at least one acceptance criterion: Pass / Missing
- Test strategy is defined (code projects only): Pass / Missing / N/A
- Open questions are non-blocking or intentionally deferred: Pass / Missing

Verdict: Ready / Not ready
```

Mark an item `Pass` only when you can cite the specific answer or supplied passage that satisfies it, and record that citation beside the item. An item you cannot cite is `Missing`, however strongly the surrounding material seems to imply it.

If any item is `Missing`, the verdict is `Not ready`: run another round instead of drafting or accepting the final SPEC. If the user asks to draft early, decline a final SPEC and offer continued interview, clearly labeled `Discovery Notes` (not a SPEC), or a narrower scope that could pass the gate. Pause only for actual interview answers or another required user decision, authority, credentials, or external action.

## 5. Write the final SPEC only when ready

When the readiness gate passes, save `spec-interview/<slug>/SPEC.md` with this Product SPEC contract.

This document is the sole contract for everything downstream. **Every Must-priority functional requirement and every acceptance criterion in it is a mandatory completion gate:** planning must cover each one, execution must produce evidence for each one, and reporting may not classify a run `COMPLETED` while any of them is unsatisfied. Replanning may change how a gate is reached and may never change, weaken, or drop the gate itself. There is no second gate system; these two lists are it.

```markdown
# SPEC: [Name]

Goal: [the desired outcome, in one sentence]

## 1. Summary
[Concise description of what is being built and why.]

## 2. Problem statement
[The problem, who has it, and why it matters.]

## 3. Goals and success metrics
### Goals
- ...

### Success metrics
- ...

## 4. Users and stakeholders
### Primary users
- ...

### Secondary users
- ...

### Stakeholders
- ...

## 5. Scope
### In scope
- ...

### Out of scope
- ...

## 6. User journeys
- ...

## 7. Functional requirements
Use stable requirement IDs:

| ID | Requirement | Priority | Acceptance signal |
|---|---|---|---|
| FR-001 | ... | Must | ... |

## 8. Business rules
Use stable rule IDs:

| ID | Rule | Rationale |
|---|---|---|
| BR-001 | ... | ... |

## 9. Data and integrations
### Data inputs
- ...

### Data outputs
- ...

### Integrations
- ...

## 10. Constraints and assumptions
### Constraints
- ...

### Assumptions
Only include assumptions explicitly confirmed by the user.
- ...

## 11. Edge cases and failure modes
| Case | Expected behavior |
|---|---|
| ... | ... |

## 12. Security, privacy, compliance, and abuse considerations
- ...

## 13. Accessibility, localization, and usability considerations
- ...

## 14. Acceptance criteria
Make each criterion TDD-ready by referencing the functional requirement(s), business rule(s), or constraint(s) it validates. Use Given/When/Then where helpful. A mandatory rule with no criterion pointing at it is an obligation nothing verifies, so the readiness gate rejects it:

- AC-001 (FR-001): Given ..., when ..., then ...
- AC-002 (BR-001): Given ..., when ..., then ...

## 15. Test strategy
*(Include only for code projects; omit for non-code deliverables.)*
- Test levels: unit / integration / end-to-end and what each covers
- TDD or test-after, and how tests trace back to acceptance criteria
- Test data, fixtures, environments, and CI expectations
- Mandatory test cases, including error and failure paths

## 16. Validation and launch checklist
- ...

## 17. Open questions
Only include non-blocking questions. If any question blocks requirements, return to the interview instead.
- ...
```

Keep the final SPEC self-contained when it may be used in another tool or agent. Avoid references such as "as discussed above" unless their content appears in the SPEC itself. After persistence, return control to the composite root immediately so it can select the next internal stage; do not ask the user to invoke another skill or send another prompt. Do not continue only when genuine input, authority, credentials, or external action is required.

## Question design rules

- Provide options that teach tradeoffs, not generic choices.
- Always give every question a free-text path: an `Other - specify` option plus room for a note.
- Defer dependent questions to a later round, after their prerequisite is answered.
- Recommend the safest or most scope-preserving option when uncertain.
- Prefer concrete language over abstract product jargon.
- Ask about negative space: what the product must not do.
- Ask about bad data, bad actors, failed dependencies, and user mistakes.
- Ask for examples whenever a rule or workflow could be interpreted multiple ways.
- Do not invent domain facts. If a detail matters and is unknown, ask.

## Handling different user styles

If the user is brief, offer defaults but keep the hard readiness gate.

If the user is domain-expert, ask sharper business-rule and edge-case questions instead of explaining basics.

If the user is overwhelmed, keep the round small (3-5 questions) and remind them that they can answer by clicking options and adding a note.

If the user gives a large existing brief, extract what is already answered first, then build the round around the gaps only. It does not bypass the interview.
