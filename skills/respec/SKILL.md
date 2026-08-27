---
name: respec
description: Reads everything that led to the code that exists — the SDLC artifacts, the built code, and this conversation — then writes a fresh spec to rebuild that work from scratch, with the old implementation thrown away. Writes a parallel docs/sdlc-v2/ tree and never edits docs/sdlc/. Scope is one fix, one epic, one phase, or the whole app. Use when the user says "respec this", "rewrite the spec", "we should rebuild this from scratch", "start this part again", or "the code got away from us". Do NOT use to route one issue — that is /sdlc:issue.
argument-hint: "<fix | epic NN | phase NN | all>"
---

# Respec: write the spec you wish you had

**Scope:** $ARGUMENTS

The code taught you something. Write it down as a spec, not as a patch.

## What this skill is

A one-way door, on purpose. It reads the artifacts, the code, and this
conversation, then writes a new spec for a rebuild from nothing. The old
implementation is thrown away, not ported.

The new spec is worth more than a fresh run of the chain for one reason: it
carries the evidence. Every lesson in it names a review finding, an issue ID, a
build-note deviation, or a file.

It writes to `docs/sdlc-v2/`. It never edits a file under `docs/sdlc/`, and it
never writes `state.md`. The old tree stays as history.

## Preconditions

Read `docs/sdlc/state.md`. If the file is absent, tell the user this project is
not in the chain, offer `/sdlc:concept`, and stop.

No stage has to be approved.

If `$ARGUMENTS` is empty, end the message with: **What are we respec'ing — a
fix, an epic, a phase, or the whole app?** Then stop.

## Step 1 — Resolve the scope

The scope is the unit of rebuild. It says how much built code the rebuild
throws away.

| Scope | The unit |
|---|---|
| `fix` | One behaviour that is built and wrong. |
| `epic NN` | One epic, and the code it produced. |
| `phase NN` | One phase: its epics, its test plan, and its code. |
| `all` | The whole app. |

Accept a number (`01`, `2`) or a slug (`01-upload`, `upload`). For `fix`,
accept an issue ID (`ISS-3`) or the behaviour in the user's own words.

Name what you resolved in one line: "Respec of epic 02 — upload, in phase 01."
If the epic or the phase does not exist, say so and stop. Do not guess.

## Step 2 — Gather the evidence

Read three sources, in this order.

**The artifacts in scope.** `concept.md`, `architecture.md`, `phases.md`,
`phase.md`, `test-plan.md`, every epic file in scope, `issues.md`, and any ADR
the epics reference. Read all three blocks of an epic file, not the spec block
alone.

Build-note deviations and review findings are the richest evidence in the repo.
Read them before you read the code. A deviation is the plan and the reality in
one place, with the reason already written.

**The built code.** Read what the scope owns. Use the Explore subagent when the
surface is wide. The code holds what no document recorded: the real data shape,
the endpoints, the edge cases somebody handled at 2am.

**This conversation.** Decisions and complaints from this session that never
reached a file count as evidence. Attribute them as "this session".

## Step 3 — Keep, change, drop

Sort every behaviour in scope into one bucket.

| Bucket | Meaning |
|---|---|
| **Keep** | The behaviour is right and a test proves it. It goes into the new spec unchanged. |
| **Change** | The behaviour is right, the shape hurt. Name what the rebuild does differently. |
| **Drop** | Nobody wanted it. Say what it cost. |

Every row carries evidence: a review finding, an issue ID, a build-note
deviation, or a file and a line.

A row with no evidence is an opinion. Say that it is an opinion, or cut it.

Bad: "The upload path is messy."
Good: "Change — upload retries live in three places. Review 2026-07-12, finding
2, and `app/upload.py:88`."

## Step 4 — Ask, only if you must

Two questions at most, one per message, each the last line of its message. See
[ASKING.md](../status/ASKING.md).

Three cases justify a question:

1. The evidence does not say why the rebuild is wanted.
2. Two readings of the scope are equally supported by the evidence.
3. The code holds a behaviour that no document asked for, and you cannot tell
   whether somebody wanted it.

Nothing else justifies one. If the evidence answers it, write.

## Step 5 — Write

Every file goes under `docs/sdlc-v2/`, in the same layout as `docs/sdlc/`. See
[ARTIFACTS.md](../status/ARTIFACTS.md).

| Scope | What you write |
|---|---|
| `fix` | `fixes/respec-<slug>.md` |
| `epic NN` | `phase-NN-<slug>/epic-NN-<slug>.md`, the spec block only |
| `phase NN` | the whole `phase-NN-<slug>/`: `phase.md`, `test-plan.md`, one epic file per slice |
| `all` | `concept.md`, `architecture.md`, `phases.md`, and one folder per phase already planned |

Write `docs/sdlc-v2/README.md` in every case.

An epic file gets the spec block alone. Write no build note and no review
block. Those belong to a build that has not happened.

Keep the numbers and the slugs from `docs/sdlc/`. A respec of epic 02 stays
epic 02. The tree is a replacement, not a renumber.

### The architecture rule

At scope `all`, `architecture.md` is rewritten. The architecture in scope goes.

At `fix`, `epic`, and `phase`, it is not rewritten. The architecture outside the
scope stands, and every new file names it under **Inherited, not rebuilt**. A
respec of one epic does not get to move the database.

### The header every respec file carries

```markdown
**Respecs:** <the docs/sdlc/ path this file replaces>
**Written from:** <short commit sha> on <YYYY-MM-DD>
**Throw away:** <the files and modules the rebuild deletes>
**Inherited, not rebuilt:** <the architecture and the phases outside this scope>

## Lessons from the first build

<One line per lesson, each with its evidence: a review finding, an issue ID, a
build-note deviation, or a file. No lesson without evidence.>
```

Below the header, use the template that
[ARTIFACTS.md](../status/ARTIFACTS.md) already defines for that file:
`concept.md`, `architecture.md`, `phases.md`, `phase.md`, `test-plan.md`, and
the epic spec block. Do not invent a second format for a file the chain already
defines. The `fix` file is the one new template, below.

### Two rules for the text

**Specify the behaviour, not the shape of the old code.** Never port. A spec
that describes the implementation you throw away rebuilds that implementation.

Bad: "Rewrite `UploadManager` so the retry queue is separate."
Good: "AC-2 — Given a failed upload, when the user retries, then the file
uploads once and no duplicate row appears."

**Keep to the budget.** The fix file is under 300 words. An epic file is under
600. `architecture.md` is under 900.

## Step 6 — Report

Under twenty lines, in this order:

1. The scope, as you resolved it.
2. The counts: kept, changed, dropped.
3. The files you wrote, by path.
4. The one thing the rebuild must do differently.

Do not paste the documents into the chat.

## docs/sdlc-v2/

```
docs/sdlc-v2/
  README.md                 what this tree is, and what it replaces
  concept.md                scope all
  architecture.md           scope all
  phases.md                 scope all
  phase-01-<slug>/
    phase.md                scope phase
    test-plan.md            scope phase
    epic-01-<slug>.md       scope epic and scope phase, spec block only
  fixes/
    respec-<slug>.md        scope fix
```

`docs/sdlc-v2/README.md`:

```markdown
# Rebuild spec — <project name>

**Scope:** <fix / epic NN / phase NN / all>
**Written from:** <short commit sha> on <YYYY-MM-DD>
**Replaces:** <the docs/sdlc/ paths this tree supersedes>

<Two to four sentences. Why the rebuild, and what the first build taught.>

The files under `docs/sdlc/` are history. Nothing in this tree changes them.
```

The fix template:

```markdown
# Respec — <the behaviour in one sentence>

<the shared header block>

## Behaviour

<Given / When / Then, numbered AC-1. One behaviour, or two at most.>

## What the first build got wrong

<Two lines at most, with the file and the evidence.>

## Proved by

<The test that proves the rebuild. Name it.>

## Delete

<The files and the functions the rebuild removes.>
```

## Close

Print the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). This skill ends
the chain's normal flow, so recommend the read, not a build:

```
Next
  → /sdlc:status           see where the old tree stands
    /sdlc:respec all       widen the rebuild
```

Put the uncommitted line above the block, as
[NEXT-STEPS.md](../status/NEXT-STEPS.md) requires.

## What this skill does not do

- It does not edit any file under `docs/sdlc/`.
- It does not write `state.md`, change a stage status, or approve anything.
- It does not delete code, write code, or commit.
- It does not start the rebuild.
- It does not route one issue. That is `/sdlc:issue`.
- It does not rewrite the architecture outside its scope.

## Reference

- [ASKING.md](../status/ASKING.md) — one question, last line, nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the templates every respec file uses,
  and the `docs/sdlc-v2/` layout.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the status words this skill
  reads and never writes.
- [issue/SKILL.md](../issue/SKILL.md) — the router for one issue, when the
  problem is smaller than a rebuild.
