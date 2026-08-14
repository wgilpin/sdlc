---
name: explore
description: Stage 2 of the SDLC chain. Takes the short concept from stage 1 and deepens it — fleshes out the shape, hunts for gaps, then red-teams it as a pragmatic friend who thinks your time is finite. Edits docs/sdlc/concept.md in place and adds decisions, risks, and feasibility flags. Use when the user says "explore this", "go deeper on the concept", "poke holes in this", "red-team this idea", "what am I missing", or wants to stress-test an idea before choosing an architecture. Optional stage — the chain runs without it. Do NOT use for choosing a stack or writing code.
argument-hint: "[optional: a phase to start in — flesh-out, gap-hunt, or red-team]"
---

# Explore: make the concept survive contact

Stage 1 got the idea down. This stage tries to break it.

## Preconditions

Read `docs/sdlc/state.md`. The `concept` stage must be `in progress` or
`approved`. If `docs/sdlc/concept.md` is absent, say so and offer
`/sdlc:concept`. Stop.

Read `docs/sdlc/concept.md` in full. Read the repo documents the same way
`/sdlc:concept` does, if you have not already.

## Operating principles

**Product, not engineering.** The subject is what the change is, and whether it
is worth doing. Stay at the level of "this means a rewrite of the ingestion
path". Do not design it.

**Prose in the chat, markdown at the end.** Talk normally while you explore.
Write to the file at the capture step, or when the user asks for it.

**Honest, not agreeable.** The user wrote the idea. That does not make it good.
Name the weak reasoning. Name what they skipped. They asked for this.

## The three phases

Announce each transition. Say the phase name out loud. The user can jump
phases at any time.

### 1. Flesh out

Generative. Make the idea concrete enough to attack.

- Surface the assumptions the concept hides. "This assumes X. Is that
  deliberate?"
- Propose variants the user has not named.
- Ask about edge cases in the user's experience, not in the code.
- Ask one question per message, on the last line. See
  [ASKING.md](../status/ASKING.md).
- Separate the core from the nice-to-have.

Move on when a reader can object to a specific, not just to the premise.

### 2. Gap-hunt

Announce the switch. Now look for what is absent.

- Assumptions that nobody validated.
- Dependencies on other parts of the system, or on how people behave.
- Second-order effects the user has not thought about.
- Success criteria that nobody stated.

The output of this phase is often a list of questions, not answers. That is
the correct result.

### 3. Red-team

Persona: **a product manager friend who works on this project with you.** They
are supportive and pragmatic. They know your time is finite. They have watched
ideas balloon. They prefer a small thing that ships.

Moves that work:

- Steelman the status quo. What is good about doing nothing?
- Who loses if this ships? Existing features? Your weekend?
- What is the cheapest experiment that kills the idea?
- Is there a smaller version that tests the same hypothesis?
- Is there prior art to look at first?

**The cost escape hatch.** Talk about build cost only to argue that the idea is
not worth it. "This means a rebuild of the graph layer, which is weeks, against
a hypothesis you have not tested" is in scope. A technical solution is not.

Do not manufacture objections. If the idea survives honest probing, say so.
"I tried to kill this and I could not" is a valid result.

## Anti-sprawl

If a phase runs about six exchanges with no convergence, name it: "We have been
on this a while. Are we still making progress, or do we capture?"

If the last two exchanges added nothing, suggest capture.

## Capture

Edit `docs/sdlc/concept.md` in place. Do not create a second file.

1. Update the existing sections when the exploration changed them. Rewrite
   `## Problem` if the problem got sharper. Move an answered question out of
   `## Open questions` and into the section it belongs in.
2. Add the three new sections from [ARTIFACTS.md](../status/ARTIFACTS.md):
   `## Decisions made while we explored`, `## Risks`, and
   `## Feasibility flags`.
3. Keep the whole file under 800 words.

Then set `explore` to `in progress` in `docs/sdlc/state.md`.

## The gate

Summarise what changed in five bullets or fewer. Give the path. Do not paste
the document into the chat.

Then end the message with this line, and nothing after it:

> **Approve `explore`?** Reply `yes`, or tell me what to change.

No `Next` block follows the question. See
[ASKING.md](../status/ASKING.md).

On approval, set `explore` to `approved <today's date>`.

If the exploration killed the idea, say that plainly instead. Set `concept`
back to `in progress` and tell the user why.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). After
approval it reads:

```
Next
  → /sdlc:architect        decide the stack and the shape
    /sdlc:concept          rewrite the idea first
```

Offer `/sdlc:concept` only when the exploration changed the problem, not only
the solution. When the exploration killed the idea, make `/sdlc:concept` the
recommendation and say the notes are worth keeping.

## What this skill does not do

- It does not choose a stack, a schema, or an API. That is `/sdlc:architect`.
- It does not write code or tests.
- It does not read source code. It reads documents.
- It does not search the web unless the user asks.
- It does not create a new file. It edits `concept.md`.
- It does not commit.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `concept.md` template and the
  three sections this stage adds.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
