# The state file

`docs/sdlc/state.md` records where the project sits in the chain. It is
markdown, not JSON, so the user can read it and correct it by hand.

It is the single source of truth for stage status and phase status. No other
file repeats that information.

## Template

```markdown
# SDLC state

Project: <name>
Updated: <YYYY-MM-DD>

## Stack

<One line per fact. Copied from architecture.md by /sdlc:architect.
Empty until stage 3. Example:

- Language: Python 3.12
- Web: FastAPI + Jinja2 + HTMX
- Database: Postgres 16
- Tests: pytest, run with `uv run pytest`
>

## Stages

- concept: pending
- explore: pending
- architecture: pending
- design: pending
- phases: pending

## Phases

<Empty until stage 4. Then one line per phase, in order, each with its
epics nested under it:

- [ ] 01-<slug> — in progress
  - [x] epic-01-<slug> — reviewed 2026-08-14
  - [ ] epic-02-<slug> — building
- [ ] 02-<slug> — pending

A phase with no epics under it is not planned yet. Run /sdlc:plan-phase
to fill it in.
>

## Notes for a fresh session

<What a new session must know that the artifacts do not say. Keep it under
five lines. Delete a line when it stops being true.>
```

## Status words

A stage takes one of three values:

- `pending` — not started.
- `in progress` — started, not approved.
- `approved <YYYY-MM-DD>` — the user approved it.

An optional stage takes a fourth value, `skipped`. Use it for `explore` when
the user chose to skip it, and for `design` on a project with no user
interface. A `skipped` stage never blocks a later one.

A phase takes one of four values: `pending`, `in progress`, `done`, `dropped`.
Tick the checkbox only when the phase is `done`.

An epic takes one of six values: `pending`, `building`, `built`, `reviewed`,
`merged`, `blocked`. Tick the checkbox from `reviewed` onward.

`reviewed` and `merged` carry a date: `reviewed 2026-08-14`,
`merged 2026-08-15`. Those dates are the only history the chain keeps. They
tell you how fast epics actually close. No other status carries a date.

An epic that is `merged` is on the default branch. An epic that is `reviewed`
has passed but still sits on its own branch.

## The contract every skill follows

1. Read `docs/sdlc/state.md` first. If the file is absent, only `/sdlc:concept`
   continues. Every other skill stops and tells the user to run
   `/sdlc:concept`.
2. Check the preconditions in the table below. If a stage is missing, name it
   and stop. Do not guess and do not fill the gap.
3. Do the work. Update `state.md` in the same turn. Set the `Updated:` date.

## Preconditions

| Skill | It needs |
|---|---|
| `/sdlc:concept` | nothing |
| `/sdlc:explore` | `concept` is `in progress` or `approved` |
| `/sdlc:architect` | `concept` is `approved` |
| `/sdlc:design` | `architecture` is `approved` |
| `/sdlc:phases` | `architecture` is `approved` |
| `/sdlc:plan-phase` | `phases` is `approved` |
| `/sdlc:build-epic` | the epic file exists and the epic is not `merged` |
| `/sdlc:review` | the epic is `built` |
| `/sdlc:push-epic` | the epic is `reviewed` |
| `/sdlc:loop-phase` | `phases` is `approved` |
| `/sdlc:issue` | `state.md` exists |
| `/sdlc:status` | nothing |

`/sdlc:explore` and `/sdlc:design` are optional. The chain runs without them.
Neither blocks `/sdlc:phases`.

## Backtracking

The user can go back to any earlier stage. When that happens:

1. Set the earlier stage to `in progress`.
2. Set every later stage back to `pending`.
3. Tell the user which stages you reset, in one sentence.

Do not silently keep a later stage approved after an earlier one changes.

## Reading the state, not the files

Stage status lives here and nowhere else. Do not infer status from whether a
file exists. A `phases.md` on disk does not mean the phases stage is approved.
