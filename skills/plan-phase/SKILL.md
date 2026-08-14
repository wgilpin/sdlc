---
name: plan-phase
description: Stage 5 of the SDLC chain. Turns one phase into epics and a test plan. Writes docs/sdlc/phase-NN-<slug>/ containing phase.md with numbered done criteria, test-plan.md that maps every criterion to a test, and one epic file per vertical slice with Given/When/Then acceptance criteria. Flags any done criterion that no test covers. Use when the user says "plan phase 2", "break this phase into epics", "what are the epics", "write the test plan", or names a phase to plan. Do NOT use to decide the phase order — that is stage 4, /sdlc:phases.
argument-hint: "[phase number or slug — 01, or 01-ingest]"
---

# Plan a phase: epics and the tests that prove them

**Phase to plan:** $ARGUMENTS

## Preconditions

Read `docs/sdlc/state.md`. The `phases` stage must be `approved`. If it is not,
name the missing stage and stop.

Read `docs/sdlc/phases.md` and `docs/sdlc/architecture.md` in full. Read
`docs/sdlc/concept.md` for the problem it solves.

Resolve the phase from `$ARGUMENTS`. Accept a number (`01`, `2`) or a slug
(`01-ingest`, `ingest`). If `$ARGUMENTS` is empty, take the first phase in
`state.md` that is not `done`. If the phase does not exist in `phases.md`, say
so and stop. Do not guess.

## Orient on the code

Read the parts of the repo this phase touches. Use the Explore subagent when
the surface is wide. You need to know:

- Which files exist already, and which the phase adds.
- How this project writes tests, and the exact command that runs them.
- Which seams the tests can use.

Read `CONTEXT.md` and `docs/adr/` if they exist. Match their vocabulary.

## Step 1 — Done criteria

Turn the phase goal into a numbered list of observable outcomes. Each one gets
an ID: `DC-1`, `DC-2`.

A good done criterion:

- Describes what a person can observe, not what the code contains.
- Is true or false. No "works well".
- Needs no other criterion to make sense.

Bad: "The parser is robust." Good: "DC-3 — A file with a malformed row loads,
and the malformed row appears in the error list."

Five to ten criteria per phase is normal.

## Step 2 — Slice into epics

An epic is one vertical slice. Three tests, and all three must pass:

1. It changes a few files, not a layer.
2. It delivers one clear behaviour.
3. Tests can prove it.

Half a day to two days of work each. Three to six epics per phase is normal.

Rules:

- **Epic 1 is the thinnest end-to-end line** through the phase.
- **No horizontal epics.** "Add the models" is not an epic.
- Order epics so each one runs on top of the last.
- Say which epics are independent. Independent epics can run in any order.

Every done criterion belongs to at least one epic. Every epic covers at least
one done criterion. Say which, in both files.

## Step 3 — Acceptance criteria per epic

Write each acceptance criterion in Given/When/Then form, numbered `AC-1`,
`AC-2`. Three to six per epic.

Each acceptance criterion names a state, an action, and a result. A criterion
you cannot write as a test is not a criterion. Rewrite it or drop it.

## Step 4 — The test plan

Build the coverage table. One row per done criterion.

Mark each row `automated` or `manual`. Prefer automated. Use `manual` only when
no headless test can reach it — a visual layout, a real payment, a device
feature. A manual row needs the exact steps a person follows.

**Then hunt the gaps.** Any done criterion with no row is a gap. List every gap
under `## Gaps` with the reason. An empty gap list is the goal, not the
default. Do not invent a test to close a gap on paper.

Use the highest seam that still proves the behaviour. A test through the HTTP
route beats four tests on the functions behind it.

## Step 5 — Write the files

Create `docs/sdlc/phase-NN-<slug>/`. Write, with the templates in
[ARTIFACTS.md](../status/ARTIFACTS.md):

- `phase.md`
- `test-plan.md`
- One `epic-NN-<slug>.md` per epic. Write only the spec block. Stage 6 and
  stage 7 append to the same file.

Then update `docs/sdlc/state.md`:

- Set the phase to `in progress`.
- Add every epic under it with status `pending`.

## The gate

Report in this order, and keep it short:

1. The done criteria, one line each.
2. The epics, in build order, one line each.
3. The gaps. If there are none, say "No gaps."

Give the folder path. Do not paste the files into the chat.

Then end the message with this line, and nothing after it:

> **Approve `phase NN`?** Reply `yes`, or tell me what to change.

No `Next` block follows the question. See
[ASKING.md](../status/ASKING.md).

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). After
approval it reads:

```
Next
  → /sdlc:build-epic 01    build the tracer bullet
    /sdlc:phases           the phase is too big
```

Name the epic number. Recommend the first epic that is not `reviewed`.

When the gap list is not empty, make the gap the recommendation instead: say
which done criterion has no test, then end the message with: **How do we
prove `DC-N`?** Name a test, or reply `manual` to record it as a manual step.

When two epics are independent, name both and say either one can go first.

## What this skill does not do

- It does not write code or tests. It names them.
- It does not run the test suite.
- It does not change `phases.md` or `architecture.md`. If the phase does not
  fit the architecture, say so and stop.
- It does not plan more than one phase at a time.
- It does not commit.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `phase.md`, `test-plan.md`, and
  epic file templates.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
