---
name: phases
description: Stage 4 of the SDLC chain. Breaks an approved architecture into an ordered list of shippable phases, worked out with the user one question at a time. A phase leaves the app usable and is one to three weeks of work. Writes docs/sdlc/phases.md with a goal, a "works after this" line, and an explicit exclusion for each phase. Use when the user says "break this into phases", "what order should we build this", "phase this up", "plan the roadmap", or has an approved architecture and needs a build order. Do NOT use to break a single phase into epics — that is stage 5, /sdlc:plan-phase.
argument-hint: "(none)"
---

# Phases: decide the build order

The question this stage answers: what do I build first, and what does it let
me do?

## Preconditions

Read `docs/sdlc/state.md`. The `architecture` stage must be `approved`. If it
is not, name the missing stage and stop.

Read `docs/sdlc/concept.md` and `docs/sdlc/architecture.md` in full.

## What a phase is

A phase is a coherent slab of product. Three tests, and all three must pass:

1. **It leaves the app usable.** After the phase, you can do something you
   could not do before. A phase that only adds a database table fails this
   test.
2. **It is one to three weeks of work.** Shorter, and it is an epic. Longer,
   and it hides too much risk.
3. **It depends only on earlier phases.** Never on a later one.

Four to seven phases is normal for a personal project. Twelve is a sign the
phases are epics.

## The rules you enforce

- **Phase 1 is a tracer bullet.** It goes end to end through every layer, in
  the thinnest possible line. One route, one handler, one query, one template,
  one test. It proves the stack works together.
- **No horizontal phases.** "Build all the models" is not a phase. "Build all
  the pages" is not a phase. Each phase cuts vertically.
- **The riskiest thing comes early.** Put the hard part from
  `architecture.md` in phase 1 or phase 2. Late risk is expensive risk.
- **Each phase names what it leaves out.** The exclusion is the useful half.

## The session

1. **Propose first.** Read the architecture, then put a full breakdown on the
   table. Four to seven phases, each with a title and a one-line goal. Do not
   ask the user to invent it from nothing.
2. **Then grill it.** Ask one question per message. Wait for the answer.
   Always give your own recommended answer with the question. The question is
   the last line of the message, as [ASKING.md](../status/ASKING.md) sets out.

Questions that earn their place:

- "Phase 2 needs the search index that phase 4 builds. Do we move the index up,
  or does phase 2 use a slower query for now?"
- "After phase 1 you can add a document but not read it back. Is that usable
  enough, or does phase 1 need the read path too?"
- "Phase 3 is two weeks of auth work. Do you need it before the app is
  private, or does a single hard-coded user carry you to phase 5?"

3. **Stop when the order holds.** The test: you can say what works after each
   phase, and no phase needs a later one.

If a question exposes a hole in the architecture, say so, then end the message
with: **Go back to `/sdlc:architect`?** Reply `yes`, or `carry on` to note it
as an open question. Do not patch the architecture from here.

## Anti-sprawl

If the discussion runs about six exchanges with no convergence, name it, then
end the message with: **Write the breakdown now and mark the disputed part as
an open question?** Reply `yes`, or keep going.

## Writing the document

Write `docs/sdlc/phases.md` with the template in
[ARTIFACTS.md](../status/ARTIFACTS.md).

Rules:

- One section per phase, in build order.
- Every phase has a goal, a "works after this" line, a "not in this phase"
  list, and its dependencies.
- Status does not go in this file. Status lives in `state.md`.
- Aim for under 100 words per phase.

Then update `docs/sdlc/state.md`:

- Add every phase to `## Phases`, in order, each with status `pending`.
- Set `phases` to `in progress`.

## The gate

List the phase titles and their one-line goals. Nothing else. Give the path.

Then end the message with this line, and nothing after it:

> **Approve `phases`?** Reply `yes`, or tell me what to change.

No `Next` block follows the question. See
[ASKING.md](../status/ASKING.md).

On approval, set `phases` to `approved <today's date>`.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). After
approval it reads:

```
Next
  → /sdlc:plan-phase 01    epics and tests for phase 1
    /sdlc:architect        the architecture has a gap
```

Always name the phase number in the command. Recommend `/sdlc:plan-phase` for
the first phase that is not `done`, which is normally `01`.

Plan one phase at a time. Do not offer to plan them all.

## Re-phasing later

The user can come back after a phase ships. That is normal and healthy.

To add a phase, give it the next free number and put it in the right place in
the ordered list. Numbers never change once written. To drop a phase, mark it
`dropped` in `state.md` and say why in `phases.md`. Do not delete the section.

## What this skill does not do

- It does not break a phase into epics. That is `/sdlc:plan-phase`.
- It does not write tests or a test plan.
- It does not estimate in days. It sizes in weeks, roughly.
- It does not change `architecture.md`.
- It does not write code.
- It does not commit.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `phases.md` template.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
