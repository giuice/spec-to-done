# spec-to-done

One Skill that takes substantial work from an unclear request to a verified outcome:

```text
SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT
```

`spec-to-done` keeps the entire workflow inside one continuous, resumable process. It interviews before planning, verifies work instead of assuming it is complete, adapts the plan when reality changes, and finishes with a concise report of what became true.

Português: [README.pt-BR.md](README.pt-BR.md).

## Why

A good plan does not guarantee a good execution.

Planning modes are genuinely useful, but on longer work the failure happens after the plan: assumptions creep in, the implementation diverges from the original intent, the plan goes stale, context is lost, and the agent eventually says *done* for something that is not what was asked.

`spec-to-done` is not another planning prompt. It is the protocol around the work:

- the SPEC is the outcome contract, and it is only written after a readiness gate passes;
- the plan contains future work only;
- every task carries an observable `Done when` condition and is verified independently of whoever performed it;
- execution evidence is appended to a durable track instead of living in the conversation;
- after every task a replan gate asks whether the remaining plan is still true;
- replanning may change future work, but it cannot silently weaken the agreed outcome.

## Install

Install through [skills.sh](https://skills.sh):

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

The installer lets you choose the compatible agent environments where the Skill should be installed.

## Using it

The workflow is close to hands-off. You describe the outcome, answer one or more interview rounds, and the Skill drives the remaining stages by itself, stopping only when it genuinely needs you.

### 1. Ask for the outcome

Natural language, no stage names required:

```text
Use spec-to-done to take our support-inbox auto-triage from idea to done.
```

### 2. Answer the interview

This is the one part that needs your hands.

The Skill asks the whole round at once — typically 7 to 10 questions covering goals, users, scope, business rules, constraints, edge cases, and acceptance criteria. Every question offers options that expose the tradeoff, one of them usually marked `(Recommended)`, plus a free-text path so you are never forced into a fixed choice.

Rounds are delivered in one of two ways:

- **Native question UI**, when your agent host provides one and it can carry the full round.
- **A generated HTML page**, when it cannot. The Skill writes `spec-interview/<slug>/round-N.html` — a single self-contained file with no network access. Open it in a browser, answer, click **Copy answers**, and paste the returned block into the chat. There is also a **Download answers.json** fallback.

The Skill then restates how it read each answer before continuing, so a misparse surfaces immediately.

Expect more than one round when a domain is still weak. It will not draft the SPEC while any required domain is incomplete, and it will never answer a question on your behalf.

### 3. Let it run

Once the SPEC is `Ready`, planning, execution, verification, replanning, and reporting continue automatically. You do not invoke the stages separately.

### 4. Resume whenever you want

Close the session, hit a context limit, come back next week:

```text
Use spec-to-done to continue the support-inbox auto-triage work.
```

The Skill reads the persisted artifacts, works out which stage is actually current, and picks up from evidence — not from a reconstruction of the chat history.

### 5. When it stops, it tells you why

It keeps going until the outcome is verified or it hits something that genuinely requires a human: credentials, authority, a destructive or irreversible action, a real product decision, or an ambiguity it must not resolve alone. Every ending — success, partial, blocked, or failed — goes through the reporter, so the run never stops silently.

### Optional: a single stage

If you explicitly ask for one stage only, that request sets where the workflow stops. It does not skip the stages before it: asking for a plan when no SPEC is ready runs the interview first, then the plan, and then stops.

## What it does

| Stage | Result |
|---|---|
| Specify | Interviews you until the desired outcome, scope, rules, risks, and acceptance criteria are clear. |
| Plan | Creates an executable plan containing only work that still needs to happen. |
| Execute | Performs and verifies one task at a time while recording durable evidence. |
| Replan | Adjusts future work when execution changes what the plan assumed. |
| Report | Communicates what was completed, what was verified, and anything that still requires attention. |

The workflow stores its state under `spec-interview/<work-slug>/`:

```text
state.md       lifecycle status, interview answers, coverage, and the readiness gate
round-N.html   a generated interview round (only when the host UI cannot carry it)
SPEC.md        the outcome contract
PLAN.md        future work
TRACK.md       append-only execution evidence and resume state
REPORT.md      the final developer-facing result
```

Because the state lives in these artifacts instead of only in the conversation, the work can survive interruptions, context limits, and a new session without reconstructing progress from memory.

### One active work at a time

Each work folder carries its lifecycle in the first line of `state.md`:

- `active` — the one work the Skill may advance. At most one, repository-wide.
- `frozen` — kept as documentation. The Skill does not route into it, resume it, or reuse its slug.
- `closed` — its terminal report was written and the cycle ended.

Starting new work, or naming a frozen one, makes that work `active` and freezes the previous one — stated, never silent. Ask to freeze a work at any time to park it without losing its artifacts. `closed` is not the same as `COMPLETED`: a work can close as blocked or failed.

## Designed for long-running work

- Starts substantial work with specification, even when the initial brief sounds detailed.
- Routes trivial, reversible changes away from unnecessary ceremony.
- Keeps completed work out of the active plan and preserves its evidence in `TRACK.md`.
- Verifies each task against observable state before advancing, rather than trusting the performer's own report.
- Distinguishes *implemented*, *verified*, *attested*, and *not verified*, and never presents unverified work as complete.
- Replans only future work; it never silently weakens the agreed outcome.
- Reconciles interrupted writes between execution, tracking, gates, and planning.
- Can resume the same workflow after a previously exhausted blocker is demonstrably resolved.
- Pauses for authority, credentials, destructive actions, or genuine product decisions instead of guessing.

## Compared with plan and goal modes

They solve different halves of the problem, and `spec-to-done` is not a replacement for either:

> Plan mode plans. Goal mode persists. `spec-to-done` governs the whole path from ambiguity to verified completion.

## When to use it

Use `spec-to-done` for features, products, migrations, integrations, redesigns, launches, or other outcomes with multiple stages, important constraints, or meaningful failure modes.

Skip it for a tiny reversible change with one obvious outcome, such as changing a button color, or when you explicitly want only one isolated stage.

## Package

The Skill is self-contained in [`skills/spec-to-done`](skills/spec-to-done):

```text
spec-to-done/
├── SKILL.md
├── assets/interview-round.template.html
└── references/{specify,plan,execute,report}.md
```

It has no Python runtime dependency and no provider-specific configuration.

## Feedback

The project is experimental and the most useful reports come from real work — a feature, a migration, an integration, a refactor — rather than a toy task. Criticism is welcome, particularly on:

- whether the specification phase is too strict;
- whether the added ceremony pays for itself on long-running work;
- whether the track and replan model actually helps after an interruption;
- whether it catches drift that a normal plan or goal workflow would have missed.

Open an issue at [giuice/spec-to-done](https://github.com/giuice/spec-to-done).

## License

[MIT](LICENSE) © 2026 giuice
