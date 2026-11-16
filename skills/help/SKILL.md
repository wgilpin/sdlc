---
name: help
description: Describes the SDLC chain — the eight stages, what each one writes, the rules the chain follows, and where to start. Use when the user asks "what is sdlc", "how does this work", "what skills are there", "sdlc help", "explain the sdlc chain", or names a stage and asks what it does. Do NOT use to report a project's current position — that is /sdlc:status.
argument-hint: "[stage name — concept, architect, build-epic] (blank = the whole chain)"
---

# Help: what this system is

**Asked about:** $ARGUMENTS

Explain, then offer where to go next.

## What this skill is

A read-only description of the chain. It never reads project files, never
writes a file, and never starts a stage until the user picks one from the
closing menu.

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

  -  /sdlc:loop-phase   stages 5 to 8, one gate, no stops
  -  /sdlc:issue        route one issue: fix now, or file it
  -  /sdlc:respec       rewrite the spec, throw the code away
  -  /sdlc:status       where you are, what runs next
  -  /sdlc:help         this

Stages 5 to 8 repeat for every epic. Stages 4 to 8 repeat for every phase.
Stages 2 and 3b are optional. The chain runs without them. Stage 3b applies
only to a project with a user interface.

/sdlc:loop-phase runs stages 5 to 8 for a whole phase. It asks once, then
builds, reviews, and lands every epic without another stop.

/sdlc:issue takes one thing that is wrong and works out where it belongs. If
the behaviour is built, it offers to fix it now. If it is still ahead, it adds
it to the epic or the phase that owns it. Every issue is logged in issues.md.

/sdlc:respec is for a problem bigger than an issue. It reads the artifacts, the
code, and the session, then writes a fresh spec to rebuild from scratch. Scope
is one fix, one epic, one phase, or the whole app. It writes to docs/sdlc-v2/
and changes nothing under docs/sdlc/.
```

## The rules, in one block

Print this after the overview.

```
Gates       Stages 1 to 5 end with "Approve <stage>, or what should change?"
            Nothing advances until you answer. Stage 6 confirms the task
            list once, then runs to green without a stop. /sdlc:loop-phase
            folds the gates of stages 5 to 8 into one question, up front.

Git         /sdlc:plan-phase drafts on plan/phase-NN-<slug> (in a worktree
            when phases.md is committed) and merges locally on approval.
            Each epic gets its own branch, epic-NN-<slug>, and /sdlc:push-epic
            commits, pushes, and merges it after asking once. No force push,
            no history rewrite, no merge of a failed review.

Order       Every skill checks its preconditions. If an earlier stage is not
            approved, it names the missing stage and stops. No code before
            stage 6.

Asking      Open questions, approval gates, and destructive checks end
            with one bold question on the last line: "Reply yes, or tell
            me what to change." Nothing follows it. Multi-option branches
            use the host's interactive question UI.

Close       A message that asks no plain-text question closes with an
            interactive next-step menu (recommended option first, with a
            plain-text fallback). Pick an option to run it directly.

Names       /sdlc:<skill> is the Claude Code plugin command. In harnesses
            without a plugin prefix (like Antigravity), invoke <skill>
            directly by name.
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
  docs/adr/NNNN-<slug>.md   decisions that are hard to reverse
  docs/sdlc-v2/             /sdlc:respec, a spec to rebuild from scratch
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
- It does not start a stage unless the user clicks one in the closing menu.
- It does not write a file.

## Close

Invoke the interactive next-step menu from
[NEXT-STEPS.md](../status/NEXT-STEPS.md), picking the recommendation from the
user's question rather than reading a state file:

New project, or the user asked where to start:

- `(Recommended) /sdlc:concept — get the idea down`
- `/sdlc:status — check existing project`

Existing project, or general overview:

- `(Recommended) /sdlc:status — see project position`
- `/sdlc:help concept — inspect one stage`
