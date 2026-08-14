# spec-to-done

One Skill that takes substantial work from an unclear request to a verified outcome:

```text
SPECIFY → PLAN → EXECUTE ↔ REPLAN → REPORT
```

`spec-to-done` keeps the entire workflow inside one continuous, resumable process. It interviews before planning, verifies work instead of assuming it is complete, adapts the plan when reality changes, and finishes with a concise report of what became true.

Português: [README.pt-BR.md](README.pt-BR.md).

## Install

Install through [skills.sh](https://skills.sh):

```bash
npx skills add giuice/spec-to-done --skill spec-to-done
```

The installer lets you choose the compatible agent environments where the Skill should be installed.

## Use

Ask for an end-to-end outcome in natural language:

```text
Use spec-to-done to take our support-inbox auto-triage from idea to done.
```

Or resume work from a later session:

```text
Use spec-to-done to continue the support-inbox auto-triage work.
```

You do not need to invoke the stages separately. The Skill inspects the persisted state, selects the correct stage, completes it, and continues automatically until it either reaches a verified outcome or needs a real decision from you.

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
SPEC.md    the outcome contract
PLAN.md    future work
TRACK.md   append-only execution evidence and resume state
REPORT.md  the final developer-facing result
```

Because the state lives in these artifacts instead of only in the conversation, the work can survive interruptions, context limits, and a new session without reconstructing progress from memory.

## Designed for long-running work

- Starts substantial work with specification, even when the initial brief sounds detailed.
- Routes trivial, reversible changes away from unnecessary ceremony.
- Keeps completed work out of the active plan and preserves its evidence in `TRACK.md`.
- Verifies each task before advancing.
- Replans only future work; it never silently weakens the agreed outcome.
- Reconciles interrupted writes between execution, tracking, gates, and planning.
- Can resume the same workflow after a previously exhausted blocker is demonstrably resolved.
- Pauses for authority, credentials, destructive actions, or genuine product decisions instead of guessing.

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

## License

[MIT](LICENSE) © 2026 giuice
