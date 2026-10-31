---
name: build-epic
description: Stage 6 of the SDLC chain. Builds one epic test-first and runs to green without stopping. Reads the epic file and the architecture, confirms the task list once, then loops red-green-refactor over vertical slices until the full test suite passes. Appends a build note to the epic file with the tasks done, the tests added, the suite result, and any deviation from the plan. Never commits. Use when the user says "build epic 2", "implement this epic", "start building", "make it green", or names an epic to build. Do NOT use to plan the epic — that is stage 5, /sdlc:plan-phase.
argument-hint: "[epic number or slug — 01, or 01-upload]"
---

# Build an epic: red, green, refactor, to done

**Epic to build:** $ARGUMENTS

## Preconditions

Read `docs/sdlc/state.md`. Find the epic. It must exist and must not be
`reviewed`. If the epic file is absent, name the phase and offer
`/sdlc:plan-phase`. Stop.

Read, in full:

- The epic file `docs/sdlc/phase-NN-<slug>/epic-NN-<slug>.md`
- `docs/sdlc/phase-NN-<slug>/test-plan.md`
- `docs/sdlc/architecture.md` — the `## Stack` block gives the test command
- `docs/sdlc/design.md`, if it exists and this epic touches the UI
- `CONTEXT.md` and `docs/adr/`, if they exist

When `design.md` exists, build the UI from its tokens and its component list.
Do not invent a colour, a spacing value, or a component it does not name. If
the epic needs one that is missing, say so in the build note under
`### Deviations from the plan`.

When Stitch screens exist, `design.md` names the project under
`## Where the screens live`. Fetch a screen with `mcp__stitch__get_screen` and
apply it to the templates. For an HTMX and Jinja2 project, use the
`stitch-to-htmx` skill, so the dynamic behaviour survives the new markup.

Resolve the epic from `$ARGUMENTS`. If `$ARGUMENTS` is empty, take the first
epic in the current phase that is not `reviewed`.

If another epic in the current phase is `built` (unreviewed), stop before you
touch a branch or write code. End the message with these two blocks and nothing
after them:

> WARNING: `epic-NN` is built but has not been reviewed or merged. Skipping
> review leaves its acceptance criteria, drift, and security checks unverified.
>
> **Review `epic-NN` first?** Reply `yes` to run the three passes on `epic-NN`,
> or `skip` to build the new epic anyway.

On a `yes`, do [review](../review/SKILL.md) in full on that earlier epic and
close from there. On `skip`, continue with the requested epic.

## Check for a repository first

Run `git rev-parse --is-inside-work-tree`.

If the answer is not `true`, stop before you write any code. Tell the user this
directory has no version control, and that a build with no way back is a bad
trade. Then end the message with:
**Run `git init` here?** Reply `yes`, or `no` to build without version control.

Never run `git init` without an answer.

If the user declines, say once that you build without a safety net, then
continue. Do not raise it again in this run.

## Work on a branch

One branch per epic. Never build on the default branch.

1. Find the default branch. Read
   `git symbolic-ref --quiet refs/remotes/origin/HEAD`. When there is no
   remote, use `main` if it exists, else `master`.
2. Check the working tree. If it is dirty with changes from other work, list
   them, then end the message with: **What do I do with these changes?** Reply
   `stash`, `carry` to take them onto the epic branch, or name the files to
   drop. Do not stash without an answer.
3. If the current branch is already `epic-NN-<slug>` for this epic, stay on it
   and say so. A second run of the same epic continues its branch.
4. Otherwise create `epic-NN-<slug>` from the default branch. Pull the default
   branch first when a remote exists.

Say the branch name in your first message. The user needs to know where the
work lands.

## The one gate

Turn the acceptance criteria into an ordered task list. One task per vertical
slice. Show the list. Name the test command you will run.

Then end the message with this line, and nothing after it:

> **Start building?** Reply `yes` to run to green, or tell me what to change.

After approval, run to green without another stop. Set the epic to `building`
in `state.md`.

Every ask in this skill follows [ASKING.md](../status/ASKING.md): one question,
last line, nothing after it.

## The loop

Follow [TDD-LOOP.md](TDD-LOOP.md). In short, per slice:

1. Write the failing test. Run it. Watch it fail for the right reason.
2. Write the smallest code that passes it.
3. Run the full suite.
4. Refactor while the suite stays green.

Slice 1 is the tracer bullet: the thinnest line through every layer this epic
touches.

### Parallel tool batching

Always batch independent tool calls into a single turn rather than emitting one tool call per turn:

1. **Batch reads:** Read the epic file, `state.md`, and all existing files listed in the epic's `## Files` section in a **single parallel turn**. Do not read files one by one across separate turns.
2. **Batch writes across distinct files:** When creating or editing multiple distinct files for a slice, emit all `write_to_file` / `replace_file_content` calls for different files in a **single parallel turn**.
3. **No unrequested shell commands:** Run only the test command from `architecture.md`. Do not run extra `build`, `find`, `git status`, or cleanup commands unless diagnosing a test failure.
4. **Batch bookkeeping:** When the suite is green, append the build note to the epic file and update `docs/sdlc/state.md` in a **single parallel turn**.

## When to stop early

Stop and report at the first of these:

- **Green.** Every test passes, including the tests that existed before.
- **Stuck.** The same test fails three times in a row. Report the failure and
  what you tried. Do not delete or skip the test.
- **Conflict.** The work needs something that contradicts `architecture.md`.
  Report the contradiction. Do not change the architecture.
- **A destructive command.** The work needs `docker`, a migration, or a
  database reset. Stop the run and end the message with the hazard and the
  question: **Run `<the exact command>`?** Reply `yes`, or `no` to stop here.
  The user's rules require the ask, even mid-build.

Never mark an epic green while a test fails. Never skip a test to reach green.

## Pre-existing failures

Run the full suite before slice 1. If tests already fail, list them, then end
the message with: **Fix these first?** Reply `yes`, or `later` to build on a
red suite.

The user's rules say to fix all failing tests, not only the ones you caused. Do
not build on a red suite without saying so.

## The build note

Append the build note block from [ARTIFACTS.md](../status/ARTIFACTS.md) to the
epic file. Do not rewrite the spec block above it.

Record honestly:

- Every task done.
- Every test file and test name added.
- The exact command run, and the summary line of its output.
- Every deviation from the spec, and why. "None" is a valid answer.
- Every follow-up you chose not to do.

Then set the epic to `built` in `state.md`.

## Report

Six lines or fewer. Say: green or blocked, how many tests were added, the suite
result, and any deviation.

List the changed files so the user can commit them.

## Close

**On green, ask for the review.** A built epic is not a finished epic, and a
`Next` line is easy to walk past. End the message with the question and nothing
after it, as [ASKING.md](../status/ASKING.md) requires:

```
Uncommitted: 6 files changed. Commit before you continue.

**Review `epic-03` now?** Reply `yes` to run the three passes, or `no` to stop
here.
```

On a `yes`, do [review](../review/SKILL.md) in full — all three passes — and
close from there. On a `no`, warn the user that the review was skipped and
print the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md) in the next
message:

```
WARNING: `epic-03` has not been reviewed. Spec conformance, drift, and
security checks have not run, and `/sdlc:push-epic` will refuse to land it.

Next
  → /sdlc:review 03        check it against the spec
    /sdlc:status           see the whole project
```

The epic stays `built` either way. Only the review moves it on, and
`/sdlc:push-epic` refuses a `built` epic. Never skip the question to save a
turn.

**Inside `/sdlc:loop-phase`, do not ask.** A subagent runs this skill there.
The loop gate already covers the task list and the review. At any stop, the
subagent returns a status to the loop instead of asking. The loop's
`## The build agent` section gives the format.

On `blocked`, do not ask, and recommend nothing downstream. Say what failed,
then offer the one action that unblocks it. `/sdlc:review` never runs on a
blocked epic.

On a conflict with `architecture.md`, make `/sdlc:architect` the
recommendation and name the contradiction.

## What this skill does not do

- **It does not commit or push.** It lists the changed files and stops. The
  user commits.
- It does not build more than one epic per run.
- It does not change the epic spec, `phases.md`, or `architecture.md`.
- It does not run `docker`, `alembic`, or any infrastructure command without
  asking first.
- It does not review its own work. That is `/sdlc:review`.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [TDD-LOOP.md](TDD-LOOP.md) — the loop, the seam rules, and the mocking rules.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the build note block.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
