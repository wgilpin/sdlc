---
name: status
description: Reports where a project sits in the SDLC chain and which skill runs next. Reads docs/sdlc/state.md and reports the stage, the current phase, the current epic, and the one command to run next. Use when the user asks "where are we", "what's next", "what stage is this project at", "sdlc status", or picks up a project after a break. Also use at the start of a fresh session on a project that has a docs/sdlc/ folder.
argument-hint: "(none)"
---

# Status: where the chain is

Report the position. Then stop.

## What this skill is

A read-only report. It answers one question: what runs next?

It never edits a file. It never starts the next stage.

## Steps

1. Read `docs/sdlc/state.md`.
2. If the file is absent, check whether `docs/sdlc/` exists.
   - No folder: tell the user this project is not in the chain. Offer
     `/sdlc:concept`. Stop.
   - Folder but no `state.md`: tell the user the state file is missing, then
     end the message with: **Rebuild `state.md` from the files present?** Reply
     `yes`, or `no`. Never write it without an answer.
3. Read the artifacts that the state says are approved. Read only their
   headings, not their bodies, unless the user asks for detail.
4. Report in the format below.

## Report format

Keep it to twenty lines or fewer.

```
Project: <name>          Updated: <date>

Stages
  concept        approved 2026-08-13
  explore        approved 2026-08-13
  architecture   approved 2026-08-14
  phases         approved 2026-08-14

Progress   8/14 epics reviewed · phase 2 of 5

  01-ingest      done          4/4
  02-search      in progress   4/6
    epic-05-rank     reviewed 2026-08-19
    epic-06-facets   building
  03-auth        pending       not planned
  04-export      pending       not planned
  05-polish      pending       not planned

Next: /sdlc:build-epic 06
```

Rules for the progress block:

- One line per phase, in order, with its status and its epic count.
- Expand the epics of the current phase only. Collapse every other phase to
  its count.
- A phase with no epics in `state.md` shows `not planned`, not `0/0`.
- The `8/14` total counts only phases that are planned. Say so in one line
  under the report when any phase is unplanned:
  `Note: phases 3 to 5 are not planned, so the total will grow.`
- Show `phase 2 of 5` only when every phase is in `phases.md`.

## Close

Replace the `Next:` line above with the block from
[NEXT-STEPS.md](../status/NEXT-STEPS.md):

```
Next
  → /sdlc:build-epic 06    build the last epic
    /sdlc:plan-phase 03    plan the next phase early
```

Use the routing table in that file to pick the recommendation. If any epic is
`blocked`, put it on its own line above the block and make it the
recommendation.

When the state and the files disagree, say so in one line under the report.
Example: `Note: phases.md exists but the phases stage is pending.` Do not fix
it yourself.

## Rebuilding a lost state file

Only when the user asks. Infer stage status from the files present, then set
every inferred stage to `in progress`, never to `approved`. Only the user
approves a stage. Say which stages you inferred and that none are approved.

Use the template in [STATE-FORMAT.md](STATE-FORMAT.md).

## What this skill does not do

- It does not run the next stage.
- It does not approve a stage.
- It does not edit `state.md`, unless the user asks it to rebuild the file.
- It does not read source code.

## Reference

- [ASKING.md](ASKING.md) — how to ask: one question, last line, nothing after
  it.
- [STATE-FORMAT.md](STATE-FORMAT.md) — the state file, its status words, the
  precondition table, and the backtracking rule.
- [ARTIFACTS.md](ARTIFACTS.md) — every file the chain writes, and its template.
