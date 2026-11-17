---
name: loop-phase
description: Runs stages 5 to 8 for a whole phase in one go. Plans the phase into epics, then loops every epic through build, review, and land without asking again. One gate at the start covers the plan, the builds, and the git writes. Stops only when a rule forces it — a red suite, a second failed review, a merge conflict, an infrastructure command, or a contradiction with the architecture. Use when the user says "loop this phase", "build the whole phase", "run phase 2 end to end", "plan and build it all", or "don't stop between epics". Do NOT use to build one epic — that is stage 6, /sdlc:build-epic.
argument-hint: "[phase number or slug — 02, or 02-search]"
---

# Loop a phase: plan it, then build, review, and land every epic

**Phase to loop:** $ARGUMENTS

This skill is stage 5, 6, 7, and 8 in one run. It asks once, then it keeps
going.

## The deal

You ask one question. After a yes, you take your own recommendation at every
step and you do not ask again. A stop costs the user their turn, so a stop
must be forced by a rule in [When a rule forces a stop](#when-a-rule-forces-a-stop).

Nothing here weakens a rule. The gates of stages 5 to 8 are not skipped — they
are answered in advance, by one question that names what the run does.

Each epic builds in a fresh subagent. The main context plans, reviews, lands,
and keeps one line per epic. A long phase does not lose detail to compaction,
and the context that reviews an epic is not the one that wrote it.

## Preconditions

Read `docs/sdlc/state.md`. The `phases` stage must be `approved`. If it is not,
name the missing stage and stop.

Read `docs/sdlc/architecture.md`, plus `AGENTS.md` and `CLAUDE.md` (if present,
for stable repo/tooling rules and test flags) and `CONTEXT.md` (if present,
strictly for domain glossary terms).

Resolve the phase from `$ARGUMENTS`, the way
[plan-phase](../plan-phase/SKILL.md) does. If `$ARGUMENTS` is empty, take the
first phase that is not `done`.

Then run these three checks, before you plan anything. Each one is a question,
in its own message, and each follows [ASKING.md](../status/ASKING.md).

1. **A repository.** Run `git rev-parse --is-inside-work-tree`. If the answer
   is not `true`, this run cannot land anything. End the message with:
   **Run `git init` here?** Reply `yes`, or `no` to stop. Never run `git init`
   without an answer. On `no`, stop the run — a loop with no way back is a bad
   trade.
2. **A clean tree.** Run `git status --short`. If changes from other work are
   present, list them, then ask what to do using the interactive question tool
   from [ASKING.md](../status/ASKING.md) (`ask_question` / `AskUserQuestion`,
   with plain-text fallback):
   - `(Recommended) Stash changes — git stash before looping`
   - `Carry changes — bring onto first epic`
   (The user can also select `Other` to name files to drop.)

3. **A green suite.** Run the full suite from the `## Stack` block in
   `architecture.md`. If tests already fail, list them, then end the message
   with: **Fix these first?** Reply `yes`, or `later` to loop on a red suite.
   The user's rules say to fix all failing tests, not only the ones you caused.

A run that starts dirty or red ends in a bad merge. Pay these three questions
up front, not one epic at a time.

## Step 1 — Plan the phase

Do the whole of [plan-phase](../plan-phase/SKILL.md): the plan branch or
worktree setup, the done criteria, the epics, the acceptance criteria, the test
plan, and the files under `docs/sdlc/phase-NN-<slug>/`. Update `state.md`.

Two changes:

- **Its gate and merge wait.** Do not run `plan-phase`'s gate or
  `## Merge the plan` yet. Step 2's loop gate covers approval, and Step 3 merges
  the plan into `<default>` once the user replies `yes`.
- **A gap does not ask.** `plan-phase` normally asks how to prove an uncovered
  done criterion. Here, list every gap above the loop gate instead. The user
  sees the holes and decides with one answer. Do not invent a test to close a
  gap on paper, and do not mark a criterion `manual` to make the list empty.

**When the phase is already planned, skip this step.** A folder with epic files
and epics in `state.md` means a run stopped earlier. Say so, say which epics
are left, and go to the gate with the remaining epics only. If an unmerged
`plan/phase-NN-<slug>` branch or worktree exists from an earlier interrupted
run, reuse it and merge it in Step 3 after the gate.

## Step 2 — The one gate

Show, in this order, and keep each line short:

1. The done criteria, one line each.
2. The gaps, or "No gaps."
3. The epics in build order, one line each, with the branch name for each.
4. The test command, and the default branch the epics land on.
5. The count: `N epics — build, review, and land each one.`

Then end the message with these two blocks, in this order, and nothing after
them:

> WARNING: This commits, pushes, and merges into `<default>` (the phase plan, if
> new, and `N` epic branches). One command does not undo it.
>
> **Run the loop for phase NN?** Reply `yes` to build, review, and land all N
> epics, or `no` to stop.

The question is the last line. Do not call the next-steps menu here; nothing
follows the question. Wait for a clear yes. A question in reply is not a yes.

This yes is the explicit permission the user's git rules require. It covers
every commit, push, and merge the run makes — including promoting confirmed,
repo-wide `Tooling / Invariant candidates` from epic build notes into
`AGENTS.md` / `CLAUDE.md` (keeping root rule files under ~50 lines, never
writing tooling rules into `CONTEXT.md`) — and nothing else. It does not carry
into a later run, and it does not cover unrelated files.

On `no`, if Step 1 drafted on `plan/phase-NN-<slug>`, leave that branch
unmerged. When Step 1 used a worktree, `<default>` in the main tree stays clean
and the draft remains in `.worktrees/plan-phase-NN-<slug>`.

## Step 3 — The loop

0. **Merge the plan first, if drafted in Step 1.** When Step 1 created or reused
   `plan/phase-NN-<slug>`, run `## Merge the plan` from
   [plan-phase](../plan-phase/SKILL.md) before cutting `epic-01` so
   `docs/sdlc/phase-NN-<slug>/` and `state.md` are committed on `<default>`.

For each epic, in build order:

1. **Build, in a subagent.** Cut the branch from the default branch yourself.
   Then launch one build subagent via `invoke_subagent` (`TypeName: "self"`) and
   stop calling tools until it reports — the loop needs its result before the
   review. See [The build agent](#the-build-agent).
2. **Review.** Do [review](../review/SKILL.md) in full, from the main context.
   Its passes 1 and 2 run in their own agents. Pass the build agent's reported
   `suite` command and summary output to Pass 1 so it skips a redundant full
   rebuild+test cycle when no code, build/config, or test files changed since
   that run. Write the review block into the epic file. Set the state from the
   verdict. Hold the detail for the digest at the end: mid-run, print the
   verdict on the epic's line and nothing more.
3. **Land.** On `pass` or `pass with follow-ups`, do
   [push-epic](../push-epic/SKILL.md) in full, with its gate answered by the
   gate above. Reuse the verified full-suite result when no code, build/config,
   or test files changed during review and the runner requires no separate
   pre-merge check, and promote any confirmed `Tooling / Invariant candidates`
   into `AGENTS.md` / `CLAUDE.md` in the same commit. Set the epic to `merged`.
   Keep the branch: the delete question does not run in a loop, and an unmerged
   branch costs nothing.
4. **Report one line**, then start the next epic.

Between epics you are on the default branch, and it is pulled. The next branch
starts from the code the last epic landed.

### The build agent

The agent does [build-epic](../build-epic/SKILL.md) in full, except that it asks
nothing and skips the pre-Slice-1 `## Pre-existing failures` suite check
(Precondition 3 and each prior merged epic already proved `<default>` is green).
The prompt names:

- The epic ID, the epic file path, and the phase folder.
- The branch it is on, and the default branch.
- `AGENTS.md` and `CLAUDE.md` (if present) for repo/tooling rules, and
  `CONTEXT.md` (if present, strictly for domain glossary terms).
- `build-epic/SKILL.md` and `build-epic/TDD-LOOP.md`, by absolute path from
  this skill's base directory.
- These rules. The task list is approved: do not show it and do not wait.
  Never commit, push, or run any git write. Never run `docker`, a migration,
  or a database reset. Write the build note and set the epic to `built` in
  `state.md` only when the suite is green.
- **Parallel Tool Batching (mandatory for fast execution, especially on `flash`):**
  1. **Orient and batch reads:** Call `read_file` on `state.md`, the epic file,
     `AGENTS.md` / `CLAUDE.md`, `CONTEXT.md`, and known small files (`<= 300`
     lines) from `## Files` (plus shared schema/type files) in a **single
     parallel turn**. For files over ~300 lines (check with `wc -l` + `rg -n`
     in one command when unsure), locate target symbols/sections with `rg -n`
     first, then batch parallel `read_file` calls with explicit line ranges
     (`<= 400` lines per chunk) instead of triggering truncation or reading in
     sequential small chunks across turns. Never guess paths with `find`.
  2. **Verify signatures and RED:** Before generating multi-test integration
     suites against new types/DTOs, verify exact existing enum/record/module
     signatures from those reads — or compile minimal unimplemented public
     type/interface stubs as part of Slice 1 RED (a fast incremental
     build/typecheck is allowed here) — so you do not guess names and burn
     compile-fix retries. Create or update the test file(s) and run a filtered
     incremental test command (`--filter` / `--no-restore`; pass `--no-build`
     only after an explicit build of the current edits) for the RED check.
  3. **Batch edits — GREEN:** Emit non-overlapping `write_file` / `edit_file`
     calls across **all** distinct implementation files in a **single parallel
     turn** (only multiple non-adjacent edits to the *same* file require
     separate turns).
  4. **Fast inner loop, one final GREEN suite:** Use filtered incremental test
     runs (`--filter` / `--no-restore`, or `--no-build` only against freshly
     built binaries) for inner-loop slices, then run the full test suite from
     `architecture.md` plus any batched headless/runtime verification scripts
     **once** at final GREEN. Aside from an incremental stub check in step 2,
     do not run redundant clean/full builds, `git status`, `rm`, or `find`
     commands unless diagnosing a failure.
  5. **Batch bookkeeping:** Append the `## Build note` (including any concise,
     repo-wide `### Tooling / Invariant candidates`) to the epic file and set
     the epic to `built` in `docs/sdlc/state.md` in a **single parallel turn**,
     then return immediately.
- The return format below.

A subagent cannot ask the user a question. At any stop, it returns instead:

```
status: green | stuck | conflict | needs-command
summary: <1-2 sentences naming the behaviour delivered and key modules touched>
tests added: <count>
files changed: <paths>
suite: <exact full-suite command and summary line>
detail: <the failing test, the contradiction, or the exact command and its hazard>
```

Act on the status:

| Status | What the main context does |
|---|---|
| `green` | Step 2, the review. |
| `needs-command` | Ask the user, as the stop table requires. On a yes, run the command, then continue the same subagent `conversationId` with `send_message`. On a no, stop the run. |
| `stuck` | Stop. See the stop table. |
| `conflict` | Stop. See the stop table. |

Check the agent's claim. A `green` with no build note in the epic file, or an
epic not set to `built`, is not green. Treat it as `stuck`.

If `invoke_subagent` is not available, build in the main context, as
[build-epic](../build-epic/SKILL.md) describes.

### The line per epic

Print one line as each epic finishes, so the user can follow the run and stop
it if they want:

```
epic-03-upload  built 7 tests · review pass · merged into main
```

The line carries the verdict alone. The findings, the follow-ups, and any
repair build go into the digest at the end of the run. A wall of review detail
between epics buries the one thing the line is for: letting the user see the
run move, and stop it.

### A failed review

A `fail` is the one place the loop repairs itself. `/sdlc:review` recommends
`/sdlc:build-epic` on the same epic, so take that recommendation once:

1. Say which acceptance criterion failed.
2. Build again, on the same branch, against that criterion only. Use a fresh
   build agent. Add the failed criterion and the review block to its prompt.
3. Review again.

A second `fail` on the same epic stops the run. Two failures mean the spec and
the code disagree about something a loop cannot settle.

Never edit the epic spec to match the code. Never soften a verdict to keep the
loop alive.

## When a rule forces a stop

Stop at the first of these. Report where the run got to, then close.

| What happened | Why it stops |
|---|---|
| The same test fails three times | `build-epic` calls it stuck. Do not skip the test. |
| The work needs `docker`, a migration, or a database reset | The user's rules require the question, even mid-run. Ask it, and continue the loop on a yes. |
| The work contradicts `architecture.md` | Only `/sdlc:architect` changes the architecture. |
| A review fails twice on one epic | See above. |
| `git merge` conflicts | `push-epic` never resolves a conflict. |
| The suite goes red before a merge | A red suite blocks the merge, whatever the review said. |
| A commit would include a file no build note names (aside from `AGENTS.md` / `CLAUDE.md` invariant promotions) | The gate covered the epics' files and confirmed invariant promotions, not unrelated files. Ask. |
| The user interrupts | Their turn always wins. |

Only the infrastructure question resumes the same run. The rest end it. Say the
epic, say the reason, and name the one command that continues:
`/sdlc:loop-phase NN` picks up the epics that are left.

Everything else is not a stop. A refactor you want, a test that needs a
fixture, a name you dislike, a second approach that looks better — the build
agent decides it and records it in the build note. Put this rule in its prompt.

## The review digest

Collect, for every epic the loop touched, five things as they happen. Write
nothing to the user until the run ends.

1. **The verdict**, and the date the review ran.
2. **What the loop fixed itself.** A `fail` that a repair build cleared: the
   acceptance criterion, and the test that proves it now.
3. **The follow-ups**, from a `pass with follow-ups`. The loop does not act on
   these.
4. **The deviations** the build note records, and any finding a `pass` verdict
   raised but did not block on.
5. **Tooling / Invariant candidates** surfaced in build notes, and whether each
   was promoted into `AGENTS.md` / `CLAUDE.md` (or discarded as volatile or
   duplicate).

The epic files already carry every review block. The digest repeats them in one
place so the user reads the phase in one sitting, and does not open five files.

Never soften a verdict, and never drop a finding because the epic merged. A
merged epic with an open finding is the exact thing the digest exists to show.

## The report

At the end of the run, print the work summary, then the fenced ledger and review
digest, in the **same message text right before calling the closing menu** — the
interactive question tool pauses the turn immediately, so the user needs the
full picture above the prompt before answering:

1. **Work summary (`### What was built`):**
   - One bullet per epic naming the user-visible behaviour delivered and the key
     files or modules touched (using each build agent's `summary` and
     `files changed`).
   - One **Situation** bullet calling out anything that affects what to do next:
     blocked or unmerged epics, manual `DC-N` walkthroughs remaining in
     `test-plan.md`, open follow-up issues (`ISS-N`), or whether merges are
     local-only vs. pushed to a remote.
2. **Fenced ledger and review digest:** Always wrap the ledger, `Reviews`, and
   `Tooling / Invariants` inside a fenced ` ```text ` code block so Markdown
   preserves every line break instead of collapsing lines into a single
   paragraph:

````markdown
### What was built — Phase 02 (search)

- **epic-01-index:** Indexed uploaded documents for full-text search (`src/search/index.py`, `tests/test_index.py`).
- **epic-02-query:** Added `/search` query route and result template (`src/routes/search.py`, `templates/search.html`).
- **epic-03-rank:** Added BM25 title/tag ranking weights (`src/search/rank.py`).
- **epic-04-cache:** Added LRU query cache invalidated on upload (`src/search/cache.py`).
- **Situation:** Stopped on `epic-05-facets` (failed review twice on `AC-3`, unmerged); `ISS-7` logged to `issues.md`; `DC-6` manual UI check still pending.

```text
Phase 02 — search. 4 of 5 epics merged into main.

  epic-01-index    built 6 tests · review pass · merged
  epic-02-query    built 4 tests · review pass · merged
  epic-03-rank     built 9 tests · review pass with follow-ups · merged
  epic-04-cache    built 5 tests · review pass · merged
  epic-05-facets   blocked · AC-3 failed review twice

Reviews
  epic-01  pass
  epic-02  pass
  epic-03  pass with follow-ups
             ISS-7 The ranking weights are hard-coded. Move them to config.
  epic-04  pass · AC-2 failed once, fixed on the same branch by
             test_cache_evicts_oldest
  epic-05  fail twice on AC-3. Not merged. The spec and the code disagree
             about what an empty facet returns.

Tooling / Invariants
  epic-03  promoted to AGENTS.md: <one-line repo-wide rule or flag>
```
````

## The follow-ups

The run does not stop to ask about them. Write one entry per follow-up to
`docs/sdlc/issues.md` as the digest is printed — the backlog route from
[issue](../issue/SKILL.md), `Status: open`, `Lands in: unplanned`, each naming
its epic. Give the ID in the digest line, so the user can find it later.

Do not fix them, do not plan them, and do not open a question about them. They
are the next run's input.

## Close

After emitting `## The report` above in your message text, immediately invoke
the interactive next-step menu from [NEXT-STEPS.md](../status/NEXT-STEPS.md) in
that same turn (never call the tool with an empty message body):

When every epic merged and another phase remains:

- `(Recommended) /sdlc:loop-phase 03 — plan and build next`
- `/sdlc:plan-phase 03 — plan next phase only`
- `/sdlc:status — see the whole project`

When the run stopped on an epic, name the epic and the reason in the message
text before calling the tool, then recommend the skill that unblocks that epic
(alongside `Stop here — inspect failure first`), never the next phase.

When the last phase merged, say the project is done in the message and offer
`(Recommended) Done — finish the run` and `/sdlc:status — view final record`.

## What this skill does not do

- It does not run without the gate. One yes, in this run, for this phase.
- It does not loop more than one phase. The next phase is a new run and a new
  question.
- It does not delete a branch, force push, or rewrite history.
- It does not land an epic that failed review.
- It does not change the epic spec, `phases.md`, or `architecture.md`.
- It does not commit a file that the epics do not touch (aside from confirmed
  `Tooling / Invariant` promotions into `AGENTS.md` or `CLAUDE.md`).
- It does not open a pull request. Use `/sdlc:build-epic` and `/sdlc:review` by
  hand when another person reviews the code.

## Reference

- [plan-phase](../plan-phase/SKILL.md) — step 1 in full.
- [build-epic](../build-epic/SKILL.md) — what the build agent does, and the
  TDD loop.
- [review](../review/SKILL.md) — step 3.2, and the three passes.
- [push-epic](../push-epic/SKILL.md) — step 3.3, and the merge sequence.
- [issue](../issue/SKILL.md) — the backlog route, for the follow-ups.
- [ASKING.md](../status/ASKING.md) — how the preconditions and the gate ask.
- [NEXT-STEPS.md](../status/NEXT-STEPS.md) — the interactive closing menu.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
