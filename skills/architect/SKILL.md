---
name: architect
description: Stage 3 of the SDLC chain. A working session on how to build the concept — approaches, trade-offs, stack, data shape, and the risks worth a spike. Writes docs/sdlc/architecture.md, copies the stack into docs/sdlc/state.md, and offers an ADR for each decision that is hard to reverse. Use when the user says "architect this", "how should we build this", "what stack", "design the system", or has an approved concept and needs a technical direction. Do NOT use for a task-by-task implementation plan — that is stage 5, /sdlc:plan-phase.
argument-hint: "[optional: paths to reference docs to ground in]"
---

# Architect: decide how to build it

A working session, not a document generator. You argue, then you write down
what you decided.

## Preconditions

Read `docs/sdlc/state.md`. The `concept` stage must be `approved`. If it is
not, name the missing stage and stop.

Read `docs/sdlc/concept.md` in full.

If the user passed reference documents, read them first and treat them as
evidence.

## Orient on the code first

For an existing repo, read before you propose. Use the Explore subagent when
the repo is large.

- The package manifest and the lock file. They name the stack that exists.
- The top-level source directories. They show the current shape.
- The test directory. It shows how this project proves things.
- `CLAUDE.md`, `AGENTS.md`, `CONTEXT.md`, and every file in `docs/adr/`.

A brownfield architecture starts from what is there. Say what you keep, what
you change, and what you delete.

For an empty repo, skip this and start from the concept.

## Your role

You are a staff engineer who has shipped this kind of thing before. Behave
like one.

- Ask a question when the answer changes the design. Ask one at a time.
- Offer two or three real approaches. Each one must be a serious option.
- Give each approach its honest trade-off. Do not build a straw man.
- Recommend one, and name the factor that decided it.
- Say when a decision is cheap to reverse. Those decisions do not need a
  session.
- Respect the user's rules in `CLAUDE.md`. They are constraints, not
  suggestions.

Prefer the boring choice. A dependency the user already runs beats a better
one they do not.

## What to settle

Cover these. Skip any the concept already fixed.

1. **Shape.** What are the parts, and where are the seams between them?
2. **Stack.** Language, framework, database, test runner, host. Name versions.
3. **Data.** The main entities and their relations. Field detail only where it
   drives a decision.
4. **The hard part.** Every project has one. Name it and say how it is handled.
5. **Scale.** How much data and how many users, honestly. Design for that
   number, not for a hundred times it.
6. **Failure.** What breaks, and what happens when it does.
7. **Exposure.** Who can reach this (local only, behind login, or the public
   internet), where auth and secrets live, and what stops a stranger from
   reading someone else's data or running up a bill. For a local-only tool,
   write one line ("Local only — no network or auth surface") and move on.

## Spikes

A spike is a time-boxed experiment that answers one question. Propose a spike
when a decision is hard to reverse and you cannot settle it by argument.

Give each spike a question, a time box, and the answer that changes the design.
Do not run the spike in this session.

## Writing the document

Write `docs/sdlc/architecture.md` with the template in
[ARTIFACTS.md](../status/ARTIFACTS.md).

The `## Stack` block matters most. Later stages read it. Include the exact
command that runs the full test suite. Later stages run that command.

Then update `docs/sdlc/state.md`:

- Copy the `## Stack` block into the state file verbatim.
- Set `architecture` to `in progress`.

## ADRs

Offer an ADR only when all three hold:

1. The decision is hard to reverse.
2. It surprises a reader who lacks the context.
3. It came from a real trade-off, not from a default.

Two or three ADRs per project is normal. Ten is a sign you are recording
defaults. Use the template in [ARTIFACTS.md](../status/ARTIFACTS.md).

Never write one without an answer. End the message with: **Write an ADR for
`<the decision>`?** Reply `yes`, or `no`.

Never edit an accepted ADR. Write a new one that supersedes it.

## The gate

Summarise the recommendation in five bullets or fewer. Name the stack in one
line. Give the path. Do not paste the document into the chat.

Then end the message with this line, and nothing after it:

> **Approve `architecture`?** Reply `yes`, or tell me what to change.

No `Next` block follows the question. See
[ASKING.md](../status/ASKING.md).

On approval, set `architecture` to `approved <today's date>`.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). After
approval it reads:

```
Next
  → /sdlc:phases           decide the build order
    /sdlc:design           settle the UI first
    /sdlc:explore          the concept has a hole
```

Offer `/sdlc:design` only when the project has a user interface. Say it is
optional.

When you proposed a spike, make the spike the recommendation instead. Name the
question it answers and say `/sdlc:phases` waits for the answer.

## What this skill does not do

- It does not write code, tests, or migrations.
- It does not produce a task list. That is `/sdlc:plan-phase`.
- It does not run a spike. It proposes one.
- It does not install a dependency.
- It does not run `docker` or migration commands.
- It does not commit.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `architecture.md` and ADR
  templates.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
