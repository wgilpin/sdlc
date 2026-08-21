---
name: issue
description: Takes an issue in one sentence and works out where it belongs. Decides whether the behaviour is already built — a defect to fix now — or still ahead, and then routes it into the epic, the phase, or the backlog that owns it. Logs every issue in docs/sdlc/issues.md. Use when the user reports something wrong or missing in the middle of a project — "the sort order should be newest first", "the login page has no error state", "this button does nothing", "log this for later". Do NOT use to plan a phase or to build an epic — those are stages 5 and 6.
argument-hint: "<the issue, in your own words>"
---

# Issue: work out where this belongs

**The issue:** $ARGUMENTS

One question, one answer, then act. Never both classify and fix in silence.

## What this skill is

A router. It takes one issue and puts it in exactly one place:

- an epic that is not built yet, as a new acceptance criterion
- a phase that has no epics yet, as a note on the phase
- the backlog, when no phase owns it
- a fix now, when the behaviour is built and wrong
- an earlier stage, when the issue contradicts the concept or the architecture

Every issue lands in `docs/sdlc/issues.md`, whichever route it takes. That file
is the record. It is not a second source of truth for stage status.

## Preconditions

Read `docs/sdlc/state.md`. If the file is absent, tell the user this project is
not in the chain, offer `/sdlc:concept`, and stop.

If `$ARGUMENTS` is empty, end the message with: **What is the issue?** Give me
one or two sentences. Then stop.

No stage has to be approved. An issue raised at stage 1 goes to the backlog.

## Step 1 — Say the issue back

Turn the user's words into one sentence that names the observable behaviour:
the state, the action, and the wrong result. Keep their vocabulary.

Example. From "ok the sort order for showing cards should be date when I
bookmarked them, most recent at top", write: "The card list sorts by the date
the user bookmarked the card, newest first."

If the issue holds two changes, split it. Take the first, and say in one line
that the second needs its own run of this skill.

## Step 2 — Find the owner

Read, in this order, and stop as soon as you can place the issue:

1. `docs/sdlc/state.md` — the current phase and the epic statuses.
2. The epic files of the current phase, and of any later planned phase.
3. `docs/sdlc/phases.md` — the phases with no epics yet.
4. `docs/sdlc/concept.md` and `docs/sdlc/architecture.md` — for a
   contradiction.

Then decide one thing: **is the behaviour built?** Do not answer from the
documents alone. Read the code that owns the behaviour. Use the Explore
subagent when the surface is wide.

An epic that is `merged` or `reviewed` is built. An epic that is `built` is
built but not landed. Everything else is not built.

## Step 3 — Pick the route

Take the first row that matches.

| What you found | Route |
|---|---|
| The behaviour contradicts `concept.md` or `architecture.md` | **Backtrack** |
| The behaviour is built, and it is wrong or missing | **Fix** |
| A `pending` epic covers this area | **Epic** |
| A planned phase covers it, but no epic does | **Phase** |
| A phase in `phases.md` covers it, and has no epics yet | **Phase** |
| Nothing covers it | **Backlog** |

A route is a recommendation, not a decision. The user answers.

### Backtrack

The issue is not an issue. It changes what the thing is, or how it is built.
Name the line in `concept.md` or `architecture.md` that it contradicts, then
recommend `/sdlc:concept` or `/sdlc:architect`. Log it in `issues.md` as
`open`, routed to that stage. Change no other file.

### Fix

The behaviour exists and is wrong. Size it with three tests. All three must
hold for a fix now:

1. It changes three files or fewer.
2. One behaviour changes, and one test proves it.
3. It needs no change to the architecture and no new dependency.

If all three hold, offer the fix now. If any fails, say which one, and make it
an epic instead: write a new epic file in the current phase folder, numbered
next, with the issue as its goal. Add it to `state.md` as `pending`.

### Epic

Append an acceptance criterion to the epic's spec block, in Given/When/Then
form, with the next free `AC` number. Add a line under `## Tests` for the test
that proves it. Reference the issue ID.

**Only when the epic is `pending`.** Never edit the spec block of an epic that
is `building`, `built`, `reviewed`, or `merged`. Stage 6 and stage 7 read that
block, and a spec that moves under them makes the review meaningless. For a
built epic, take the **Fix** route.

If the new criterion needs a done criterion the phase does not have, add it to
`phase.md` with the next free `DC` number, and add its row to `test-plan.md`.

### Phase

The phase has no epic that owns this yet. Add the issue to the phase entry in
`docs/sdlc/phases.md` under its goal, as one line beginning `Issue ISS-N:`.
`/sdlc:plan-phase` picks it up when it plans that phase. Change no epic file.

### Backlog

Nothing owns it. It lives in `issues.md` as `open` with `Lands in: unplanned`.
Say in one line what would have to exist for it to have a home — usually a
phase that is not in `phases.md` yet.

## Step 4 — Ask

Report in this order, short:

1. The issue in one sentence, as you restated it.
2. What you found: built or not built, and the file or epic that owns it.
3. The route, and what it changes. Name every file you would edit.

Then end the message with the line for the route, and nothing after it:

| Route | The last line |
|---|---|
| Fix, small | **Fix this now?** Reply `yes` to write the test and the fix, or `log` to record it for later. |
| Fix, too big | **Add this as `epic-NN-<slug>`?** Reply `yes`, or `log` to record it for later. |
| Epic | **Add this to `epic-NN-<slug>` as AC-N?** Reply `yes`, or tell me where it belongs. |
| Phase | **Record this against phase NN?** Reply `yes`, or tell me where it belongs. |
| Backlog | **Log this to the backlog?** Reply `yes`, or name the phase it belongs to. |
| Backtrack | **Take this back to `architecture`?** Reply `yes`, or `log` to record it for later. |

See [ASKING.md](../status/ASKING.md). No `Next` block follows a question.

## Step 5 — Act

Only after a `yes`.

Write the entry in `docs/sdlc/issues.md` first, in every case. Create the file
from the template below if it is absent. Give the issue the next free ID.

Then do the route's own work. For a fix now:

1. Make sure that the working tree is clean. If it is not, say what is
   uncommitted and stop.
2. Work on the current branch. Do not cut a branch and do not commit. A fix
   found mid-epic belongs to that epic's branch and lands with it.
3. Write the failing test first. Run it. Show that it fails for the stated
   reason.
4. Make it pass. Change nothing else.
5. Run the full suite. If the suite goes red, revert your change and report.
6. Set the issue to `fixed <date>` in `issues.md`, and name the test.

Update `docs/sdlc/state.md` only when the route added an epic. Set the
`Updated:` date whenever you write to `state.md`.

## issues.md

```markdown
# Issues — <project name>

One line per issue, newest last. Status is `open`, `scheduled`, `fixed`,
or `dropped`. This file records issues. It never records stage status.

## ISS-N — <the issue in one sentence>

**Raised:** <YYYY-MM-DD>
**Status:** open
**Lands in:** <epic-NN-<slug> / phase NN / unplanned / fixed on <branch>>

<Two lines at most. What is wrong, and how you know. Name the file or the
epic. Delete nothing when the issue closes — set the status instead.>
```

An issue keeps its number for good. A dropped issue stays in the file with
`Status: dropped` and one line of reason.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). Pick the
recommendation from the route:

Fixed now, mid-epic:

```
Next
  → /sdlc:build-epic 06    finish the epic
    /sdlc:issue            report the next one
```

Routed into a pending epic:

```
Next
  → /sdlc:build-epic 06    build the epic with the new AC
    /sdlc:status           see the whole project
```

Logged to the backlog:

```
Next
  → /sdlc:status           see where this project is
    /sdlc:phases           the backlog needs a phase
```

When the fix changed code, put the uncommitted line above the block, as
[NEXT-STEPS.md](../status/NEXT-STEPS.md) requires.

## What this skill does not do

- It does not commit, branch, or merge.
- It does not edit the spec block of an epic that is `building` or later.
- It does not change stage status, and it never approves a stage.
- It does not fix more than one issue in a run.
- It does not rewrite `concept.md` or `architecture.md`. It names the
  contradiction and stops.
- It does not plan a phase. That is `/sdlc:plan-phase`.

## Reference

- [ASKING.md](../status/ASKING.md) — one question, last line, nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the epic, phase, and test plan
  templates this skill appends to.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the status words and the
  backtracking rule.
- [TDD-LOOP.md](../build-epic/TDD-LOOP.md) — the red-green-refactor loop the
  fix route follows.
