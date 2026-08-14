---
name: help
description: Describes the SDLC chain — the eight stages, what each one writes, the rules the chain follows, and where to start. Use when the user asks "what is sdlc", "how does this work", "what skills are there", "sdlc help", "explain the sdlc chain", or names a stage and asks what it does. Do NOT use to report a project's current position — that is /sdlc:status.
argument-hint: "[stage name — concept, architect, build-epic] (blank = the whole chain)"
---

# Help: what this system is

**Asked about:** $ARGUMENTS

Explain. Then stop.

## What this skill is

A read-only description of the chain. It never reads project files, never
writes a file, and never starts a stage.

Two modes:

- No argument: print the overview below.
- A stage name: print that stage's row, then read that skill's `SKILL.md` and
  give three or four lines on what it asks and what it writes.

If the user wants to know where their own project sits, point them at
`/sdlc:status` instead.

## The overview

Print this. Keep the shape.

```
Personal SDLC — one chain from a half-formed idea to reviewed code.

Each stage writes a file to docs/sdlc/ in your repo. A state file records
where you are, so a fresh session continues from disk alone.

  1  /sdlc:concept      six questions, one at a time  →  concept.md
  2  /sdlc:explore      gap-hunt and red-team it      →  concept.md (deepened)
  3  /sdlc:architect    stack, shape, trade-offs      →  architecture.md
 3b  /sdlc:design       surfaces, tokens, screens     →  design.md
  4  /sdlc:phases       shippable build order         →  phases.md
  5  /sdlc:plan-phase   epics and a test plan         →  phase-NN-<slug>/
  6  /sdlc:build-epic   red, green, refactor          →  code, on a branch
  7  /sdlc:review       spec, drift, correctness      →  review block
  8  /sdlc:push-epic    commit, push, merge, pull     →  default branch

  -  /sdlc:status       where you are, what runs next
  -  /sdlc:help         this

Stages 5 to 8 repeat for every epic. Stages 4 to 8 repeat for every phase.
Stages 2 and 3b are optional. The chain runs without them. Stage 3b applies
only to a project with a user interface.
```

## The rules, in one block

Print this after the overview.

```
Gates       Stages 1 to 5 end with "Approve <stage>, or what should change?"
            Nothing advances until you answer. Stage 6 confirms the task
            list once, then runs to green without a stop.

Git         Only /sdlc:push-epic runs a git write command, and it asks once
            first. Every other stage leaves changes in the working tree.
            Each epic gets its own branch, epic-NN-<slug>. No force push,
            no history rewrite, no merge of a failed review.

Order       Every skill checks its preconditions. If an earlier stage is not
            approved, it names the missing stage and stops. No code before
            stage 6.

Asking      A question is the last line of the message, in bold, and it
            names the answer: "Reply yes, or tell me what to change."
            Nothing follows it — no Next block, no summary. One question
            per message. If there is no question, it is your turn.

Close       A message that asks nothing ends with a Next block. The arrow
            marks the recommendation. You never work out what runs next.
```

## What it writes

Print this only when the user asks what the chain produces, or asks about
stage 5 or later.

```
docs/sdlc/
  state.md                  where you are in the chain
  concept.md                stage 1, deepened by stage 2
  architecture.md           stage 3, records the stack
  design.md                 stage 3b, tokens and surfaces, UI projects only
  phases.md                 stage 4, the ordered phase list
  phase-01-<slug>/
    phase.md                stage 5: goal, scope, done criteria
    test-plan.md            stage 5: how the phase gets proven
    epic-01-<slug>.md       stage 5 spec, stage 6 build note, stage 7 review
docs/adr/NNNN-<slug>.md     decisions that are hard to reverse
```

One file per epic. That file holds the spec, the build note, and the review,
in that order.

## Answering a question about one stage

1. Print the stage's line from the overview.
2. Read that skill's `SKILL.md`. Read the frontmatter and the headings only.
3. Say what it asks the user, what it writes, and what blocks it.
4. Name the stage before it and the stage after it.

Four to eight lines. Do not paste the skill.

## What this skill does not do

- It does not read `docs/sdlc/state.md`.
- It does not report a project's position. That is `/sdlc:status`.
- It does not start a stage.
- It does not write a file.

## Close

Use the block format from [NEXT-STEPS.md](../status/NEXT-STEPS.md), with one
change: pick the recommendation from the user's question, not from a state
file, because this skill does not read state.

New project, or the user asked what to do first:

```
Next
  → /sdlc:concept          get the idea down
    /sdlc:status           check an existing project
```

Existing project:

```
Next
  → /sdlc:status           see where this project is
    /sdlc:help concept     read one stage in detail
```
