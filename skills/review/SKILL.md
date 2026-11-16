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
- `AGENTS.md` and `CLAUDE.md` (repo/tooling rules), `CONTEXT.md` (domain
  glossary), and `docs/adr/`, if they exist

Get the diff. Prefer `git diff <default-branch>...HEAD` plus the unstaged
changes. If the repo is not a git repository, review the files named in the
build note.

## How the passes run

An agent that gets only the spec and the diff has no memory of what the builder
meant. It judges the code that exists. So passes 1 and 2 run in subagents.

1. **Launch two subagents in one parallel turn** via `invoke_subagent`
   (`TypeName: "self"`): one runs Pass 1, one runs Pass 2.
2. **Run pass 3 in the main context** while they work, then end your turn and
   wait for both subagents to report.
3. **Merge** when both agents report. See [Merging the passes](#merging-the-passes).

Each agent prompt names:

- The epic file path, the phase folder, and the paths to `architecture.md`,
  `concept.md`, and any existing `AGENTS.md`, `CLAUDE.md`, and `CONTEXT.md`.
- The diff command, with the default branch filled in.
- For Pass 1 inside `/sdlc:loop-phase`: the build agent's reported `suite`
  command and summary output, plus whether any code, build/config, or test file
  changed since that run.
- This file, by absolute path from the skill's base directory, and the one
  section the agent follows: `## Pass 1 — Conformance` or `## Pass 2 — Drift`.
- The return format for that pass, copied from the section.
- These rules: read only. Edit no file. Run no git write command. Ask the user
  nothing.

Only the pass 1 agent runs the test suite (when a suite run is needed — see
Pass 1), so concurrent agents never contend on shared test state, ports, or
databases. Pass 3 and the pass 2 agent run no tests.

If `invoke_subagent` is not available, run the three passes in sequence in the
main context. The content of each pass does not change.

## Pass 1 — Conformance

For each acceptance criterion in the epic, answer two questions:

1. Does the code do this?
2. Which test proves it?

**Verify the test run.** In a standalone review, do not trust the build note —
run the command from the `## Stack` block in `architecture.md` and read the
output. Inside `/sdlc:loop-phase`, skip re-running the full rebuild+test cycle
when all three hold: (1) the build agent ran the full suite command from
`architecture.md` (not a `--filter` subset) with 0 failures and 0 skipped
tests, (2) every acceptance criterion's test exists in the diff and asserts
real behaviour, and (3) no code, build/config, or test files changed since that
run (only `docs/sdlc/` bookkeeping). Re-run the full suite if any of the three
fails or the reported output is missing or ambiguous.

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

Hunt bugs and security exposures, in the main context. Inspect the full diff
directly (and fold in `/code-review` findings when that command is available).

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

Add the checks that `/code-review` does not know about. Read `AGENTS.md`,
`CLAUDE.md` (if present), and `docs/sdlc/architecture.md`, and enforce the
project's own rules and invariants declared there.

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

Print the phase count from `state.md` in one line (`Phase 02: 5 of 6 epics
reviewed.`) and the `Uncommitted:` reminder, then invoke the interactive
next-step menu from [NEXT-STEPS.md](../status/NEXT-STEPS.md).

On `pass`:

- `(Recommended) /sdlc:push-epic 05 — land it on main`
- `/sdlc:build-epic 06 — build next epic first`

When no pending epic remains in the phase, offer `/sdlc:status — see the whole
project` as the second option. A passing epic still sits on its own branch, so
landing it comes before starting the next epic.

On `pass with follow-ups`, print the follow-ups as a numbered list above the
`Uncommitted:` line, then offer the same menu as `pass`.

On `fail`, name the criterion that failed in the message text, then offer:

- `(Recommended) /sdlc:build-epic 05 — fix failed criteria`
- `/sdlc:status — see the whole project`

When the same epic has now failed review twice, add
`/sdlc:respec epic 05 — rewrite spec from scratch`. Never offer the next epic
after a `fail`.

## What this skill does not do

- **It does not commit or push.** It lists the changed files and stops.
- It does not fix what it finds. It reports. The user decides.
- It does not change the epic spec to match the code.
- It does not review code outside the epic's diff.
- It does not approve its own verdict. A `fail` stays a `fail`.

## Reference

- [ASKING.md](../status/ASKING.md) — plain-text questions and gates vs.
  interactive menus.
- [NEXT-STEPS.md](../status/NEXT-STEPS.md) — the interactive closing menu.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the review block.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
