---
name: review
description: Stage 7 of the SDLC chain. Reviews a built epic in three passes — conformance against the epic's acceptance criteria, drift against concept.md and architecture.md, then correctness and security. Confirms every acceptance criterion is proved by a test that actually runs. Appends a review block to the epic file with a verdict of pass, pass with follow-ups, or fail. Use when the user says "review this epic", "sdlc review", "check this against the spec", or has just finished a build. For a general bug hunt on an unrelated diff, use the built-in /code-review instead.
argument-hint: "[epic number or slug — 01, or 01-upload]"
---

# Review: does it match what we said it would do?

**Epic to review:** $ARGUMENTS

Three passes. Passes 1 and 2 run in subagents, in parallel. Pass 3 runs in the
main context at the same time.

## Preconditions

Read `docs/sdlc/state.md`. Find the epic. It must be `built`. If it is
`pending` or `building`, say so and stop.

Read, in full:

- The epic file, including its build note
- `docs/sdlc/phase-NN-<slug>/test-plan.md` and `phase.md`
- `docs/sdlc/architecture.md`
- `docs/sdlc/concept.md`
- `CLAUDE.md`, `CONTEXT.md`, and `docs/adr/`, if they exist

Get the diff. Prefer `git diff <default-branch>...HEAD` plus the unstaged
changes. If the repo is not a git repository, review the files named in the
build note.

## How the passes run

An agent that gets only the spec and the diff has no memory of what the builder
meant. It judges the code that exists. So passes 1 and 2 run in subagents.

1. **Launch two agents in one message**, both `general-purpose`, both in the
   background. One runs pass 1, one runs pass 2.
2. **Run pass 3 in the main context** while they work.
3. **Merge** when both agents report. See [Merging the passes](#merging-the-passes).

Each agent prompt names:

- The epic file path, the phase folder, and the paths to `architecture.md` and
  `concept.md`.
- The diff command, with the default branch filled in.
- This file, by absolute path from the skill's base directory, and the one
  section the agent follows: `## Pass 1 — Conformance` or `## Pass 2 — Drift`.
- The return format for that pass, copied from the section.
- These rules: read only. Edit no file. Run no git write command. Ask the user
  nothing.

Only the pass 1 agent runs the test suite. The project has one test database,
and two suites at once break each other. Pass 3 and the pass 2 agent run no
tests.

If the Agent tool is not available, run the three passes in sequence in the
main context. The content of each pass does not change.

## Pass 1 — Conformance

For each acceptance criterion in the epic, answer two questions:

1. Does the code do this?
2. Which test proves it?

**Then run the tests.** Do not trust the build note. The pass 1 agent runs the
command from the `## Stack` block in `architecture.md` and reads the output.

A criterion counts as met only when a test that passes covers it. Mark it `no`
when:

- No test names it.
- A test names it but is skipped or marked expected-to-fail.
- The test passes for a reason unrelated to the criterion.
- The test asserts a mock, not the behaviour.

A criterion marked `manual` in the test plan counts as met when the build note
records the manual walkthrough and its result. Otherwise mark it `no`.

**Return format.** One row per acceptance criterion: the criterion ID, met
`yes` or `no`, the test name, and the reason for a `no`. Then the summary line
of the suite output. The main context fills the acceptance-criteria table in
the review block from these rows.

## Pass 2 — Drift

Compare the change against the two documents above it.

- **Against `architecture.md`.** Does the code use the stack that was chosen?
  Does it respect the seams and the data shape? Does it contradict an ADR?
- **Against `concept.md`.** Does the change serve the stated problem? Did
  something in `## Out of scope` get built anyway?

Report each drift with the document, the line it contradicts, and the code that
contradicts it. Drift is not automatically wrong. Sometimes the document is out
of date. Say which you think it is, and let the user decide.

"None" is a valid and common result.

**Return format.** One line per drift: the document, the line it contradicts,
the code location, and `doc out of date` or `code wrong`. Or the one word
`None`.

## Pass 3 — Correctness and security

Hunt bugs and security exposures, in the main context. Delegate the deep pass
to the built-in `/code-review` command and fold its findings into yours.

Check every diff against these five security rules:

- **No secrets in the diff.** No hard-coded API keys, tokens, passwords,
  private keys, or `.env` values.
- **No missing auth or ownership checks.** Every non-public route or mutation
  checks authentication, and every lookup or mutation by ID verifies that the
  record belongs to the current user.
- **No unsafe input in dangerous sinks.** No string interpolation in SQL or
  shell commands, no unescaped HTML (`|safe`, `dangerouslySetInnerHTML`), and
  no file paths built directly from user input.
- **No sensitive data in logs or responses.** No passwords, tokens, session
  secrets, or raw stack traces written to logs or returned to the client.
- **No unbounded public exposure.** Any public route or webhook that calls a
  paid API or accepts uploads has a signature check, size limit, or rate limit.

Add the checks that `/code-review` does not know about. Read `CLAUDE.md` and
apply the user's own rules. In this project those include:

- No test points at the live database. Tests use a dedicated test database.
- No inline JavaScript in a template. JavaScript lives in a static file.
- No live database without scheduled backups.

Rank every finding. Say what breaks — or what an attacker can do — in plain
English, and for whom.

## Merging the passes

Wait for both agents. Then:

1. Drop any pass 3 finding that repeats a pass 1 or pass 2 finding.
2. Find the test that each pass 1 `yes` row names. If you cannot find it,
   the row is a `no`.
3. Decide the verdict below from all three passes.

Only the main context writes files. The agents report; the main context writes
the review block and `state.md`.

## The verdict

- **pass** — every acceptance criterion is met and proved. No drift. No
  finding that changes behaviour or exposes a security risk.
- **pass with follow-ups** — every criterion is met and proved. The findings
  are real but can wait. List them.
- **fail** — a criterion is not met, or not proved by a passing test, or the
  change contradicts the architecture, or Pass 3 found a security exposure.

A criterion with no test or a real security exposure (a leaked secret, missing
auth or ownership check, or injection sink) is a `fail`, not a follow-up. That
is the point of this stage.

## Writing the review

Append the review block from [ARTIFACTS.md](../status/ARTIFACTS.md) to the epic
file. Do not rewrite the spec block or the build note above it.

Then update `docs/sdlc/state.md`:

- On `pass` or `pass with follow-ups`, set the epic to
  `reviewed <today's date>` and tick it.
- On `fail`, set the epic to `blocked`.

Do not set the phase to `done`. A phase is done when every epic is `merged`,
which `/sdlc:push-epic` decides.

## Report

Lead with the verdict. Then the criteria that failed, the drift, and the top
three findings. Eight lines or fewer.

List the changed files so the user can commit them.

## Close

Give the count from `state.md` in one line, then the block from
[NEXT-STEPS.md](../status/NEXT-STEPS.md).

On `pass`:

```
Phase 02: 5 of 6 epics reviewed.

Next
  → /sdlc:push-epic 05     land it on main
    /sdlc:build-epic 06    build the last epic first
```

A passing epic still sits on its own branch. Landing it comes before the next
epic, so the next branch starts from current code.

On `pass with follow-ups`, recommend the same next epic, and put the follow-ups
above the block as a numbered list.

On `fail`, recommend `/sdlc:build-epic <same epic>` to fix what failed. Name
the criterion that failed. Never recommend the next epic after a `fail`.

## What this skill does not do

- **It does not commit or push.** It lists the changed files and stops.
- It does not fix what it finds. It reports. The user decides.
- It does not change the epic spec to match the code.
- It does not review code outside the epic's diff.
- It does not approve its own verdict. A `fail` stays a `fail`.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the review block.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
