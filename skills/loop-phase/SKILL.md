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

## Preconditions

Read `docs/sdlc/state.md`. The `phases` stage must be `approved`. If it is not,
name the missing stage and stop.

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
   present, list them, then end the message with: **What do I do with these
   changes?** Reply `stash`, `carry` onto the first epic branch, or name the
   files to drop.
3. **A green suite.** Run the full suite from the `## Stack` block in
   `architecture.md`. If tests already fail, list them, then end the message
   with: **Fix these first?** Reply `yes`, or `later` to loop on a red suite.
   The user's rules say to fix all failing tests, not only the ones you caused.

A run that starts dirty or red ends in a bad merge. Pay these three questions
up front, not one epic at a time.

## Step 1 — Plan the phase

Do the whole of [plan-phase](../plan-phase/SKILL.md): the done criteria, the
epics, the acceptance criteria, the test plan, and the files under
`docs/sdlc/phase-NN-<slug>/`. Update `state.md`.

Two changes:

- **Its gate does not run.** The loop gate below replaces it.
- **A gap does not ask.** `plan-phase` normally asks how to prove an uncovered
  done criterion. Here, list every gap above the loop gate instead. The user
  sees the holes and decides with one answer. Do not invent a test to close a
  gap on paper, and do not mark a criterion `manual` to make the list empty.

**When the phase is already planned, skip this step.** A folder with epic files
and epics in `state.md` means a run stopped earlier. Say so, say which epics
are left, and go to the gate with the remaining epics only.

## Step 2 — The one gate

Show, in this order, and keep each line short:

1. The done criteria, one line each.
2. The gaps, or "No gaps."
3. The epics in build order, one line each, with the branch name for each.
4. The test command, and the default branch the epics land on.
5. The count: `N epics — build, review, and land each one.`

Then end the message with these two blocks, in this order, and nothing after
them:

> WARNING: This commits, pushes, and merges `N` branches into `<default>`. One
> command does not undo it.
>
> **Run the loop for phase NN?** Reply `yes` to build, review, and land all N
> epics, or `no` to stop.

The question is the last line. No `Next` block, no summary. Wait for a clear
yes. A question in reply is not a yes.

This yes is the explicit permission the user's git rules require. It covers
every commit, push, and merge the run makes, and nothing else. It does not
carry into a later run, and it does not cover a file the epics do not touch.

## Step 3 — The loop

For each epic, in build order:

1. **Build.** Do [build-epic](../build-epic/SKILL.md) in full, with its task
   list confirmed by the gate above. Cut the branch, run the red-green-refactor
   loop, write the build note, set the epic to `built`. Its close does not ask
   for the review: the gate above already covers it, and step 2 runs it.
2. **Review.** Do [review](../review/SKILL.md) in full — all three passes, and
   run the tests yourself. Write the review block into the epic file. Set the
   state from the verdict. Hold the detail for the digest at the end: mid-run,
   print the verdict on the epic's line and nothing more.
3. **Land.** On `pass` or `pass with follow-ups`, do
   [push-epic](../push-epic/SKILL.md) in full, with its gate answered by the
   gate above. Set the epic to `merged`. Keep the branch: the delete question
   does not run in a loop, and an unmerged branch costs nothing.
4. **Report one line**, then start the next epic.

Between epics you are on the default branch, and it is pulled. The next branch
starts from the code the last epic landed.

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
2. Build again, on the same branch, against that criterion only.
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
| A commit would include a file no build note names | The gate covered the epics' files, not that one. Ask. |
| The user interrupts | Their turn always wins. |

Only the infrastructure question resumes the same run. The rest end it. Say the
epic, say the reason, and name the one command that continues:
`/sdlc:loop-phase NN` picks up the epics that are left.

Everything else is not a stop. A refactor you want, a test that needs a
fixture, a name you dislike, a second approach that looks better — decide it
yourself and record it in the build note.

## The review digest

Collect, for every epic the loop touched, four things as they happen. Write
nothing to the user until the run ends.

1. **The verdict**, and the date the review ran.
2. **What the loop fixed itself.** A `fail` that a repair build cleared: the
   acceptance criterion, and the test that proves it now.
3. **The follow-ups**, from a `pass with follow-ups`. The loop does not act on
   these.
4. **The deviations** the build note records, and any finding a `pass` verdict
   raised but did not block on.

The epic files already carry every review block. The digest repeats them in one
place so the user reads the phase in one sitting, and does not open five files.

Never soften a verdict, and never drop a finding because the epic merged. A
merged epic with an open finding is the exact thing the digest exists to show.

## The report

At the end of the run, print the ledger, then the digest, and nothing else:

```
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
```

Six lines of prose or fewer above the ledger. The ledger and the digest carry
the detail.

## The follow-ups

The run does not stop to ask about them. Write one entry per follow-up to
`docs/sdlc/issues.md` as the digest is printed — the backlog route from
[issue](../issue/SKILL.md), `Status: open`, `Lands in: unplanned`, each naming
its epic. Give the ID in the digest line, so the user can find it later.

Do not fix them, do not plan them, and do not open a question about them. They
are the next run's input.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md).

When every epic merged:

```
Next
  → /sdlc:loop-phase 03    plan and build the next phase
    /sdlc:status           see the whole project
```

When the run stopped on an epic, recommend the skill that unblocks that epic,
never the next phase. Name the epic and the reason in the line above the block.

When the last phase merged, say the project is done.

## What this skill does not do

- It does not run without the gate. One yes, in this run, for this phase.
- It does not loop more than one phase. The next phase is a new run and a new
  question.
- It does not delete a branch, force push, or rewrite history.
- It does not land an epic that failed review.
- It does not change the epic spec, `phases.md`, or `architecture.md`.
- It does not commit a file that the epics do not touch.
- It does not open a pull request. Use `/sdlc:build-epic` and `/sdlc:review` by
  hand when another person reviews the code.

## Reference

- [plan-phase](../plan-phase/SKILL.md) — step 1 in full.
- [build-epic](../build-epic/SKILL.md) — step 3.1, and the TDD loop.
- [review](../review/SKILL.md) — step 3.2, and the three passes.
- [push-epic](../push-epic/SKILL.md) — step 3.3, and the merge sequence.
- [issue](../issue/SKILL.md) — the backlog route, for the follow-ups.
- [ASKING.md](../status/ASKING.md) — how the three preconditions and the gate
  ask.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
