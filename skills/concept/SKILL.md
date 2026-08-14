---
name: concept
description: Stage 1 of the SDLC chain. Turns a half-formed idea into a short concept document in twenty minutes, through at most six questions asked one at a time. Writes docs/sdlc/concept.md — problem, who it is for, the smallest thing that proves the idea, out of scope, open questions — and creates docs/sdlc/state.md. Use when the user has a new app, tool, PWA, or feature idea and says "I have an idea for", "I want to build", "thinking about building", "start a new project", or "let's get this idea down". Use for a change to an existing repo too. Do NOT use for writing code or for choosing a stack — the stack is stage 3, /sdlc:architect.
argument-hint: "[the idea, in a sentence] (blank = start with questions)"
---

# Concept: get the idea down fast

**The idea:** $ARGUMENTS

Twenty minutes, not two hours. The depth pass is stage 2, `/sdlc:explore`.

## What this skill is

A short interview that produces a document you can argue with. The document is
about the problem and the smallest proof, never about how to build it.

## Before you ask anything

Read these files silently, in one pass. Do not report what you read.

- `README.md`
- Every `.md` directly in `docs/` — not the subdirectories
- `CLAUDE.md` and `AGENTS.md`
- `docs/sdlc/state.md`, if it exists

If `state.md` exists and `concept` is `approved`, tell the user a concept
already exists, then end the message with: **Which one — `deepen`, `replace`,
or `move on`?** `deepen` runs `/sdlc:explore`, `replace` overwrites
`concept.md`, `move on` runs `/sdlc:architect`.

Stop until they choose.

If the repo has code and no `README.md`, list the top-level directories and the
package manifest. That is enough to know what the project is.

## The interview

Ask at most six questions. Ask **one question per message**. Wait for the
answer before the next question. The question is the last line of the message,
as [ASKING.md](../status/ASKING.md) sets out.

Always give your own recommended answer with the question. The user corrects it
faster than they compose one from nothing.

Cover these, in this order. Skip any the user already answered.

1. **The problem.** What is wrong today? Who feels it?
2. **The person.** Who uses this? If the answer is "me", say so plainly in the
   document. A tool for one person is a real answer.
3. **Today's workaround.** What do they do now instead? If there is no
   workaround, the problem is possibly not real. Say that.
4. **The smallest proof.** What must work before you believe the idea holds?
5. **Out of scope.** What are you deliberately not building?
6. **The one thing that kills it.** What would make you stop?

Stop early when the answers converge. Six is the ceiling, not the target.

## Banned at this stage

Do not discuss, and do not let the conversation drift to:

- Language, framework, database, or hosting
- Schemas, endpoints, file names, or class names
- Libraries and packages

If the user raises one, write it in `## Open questions` and say: "That is stage
3. I noted it." Then return to the question you asked.

## Writing the document

Create `docs/sdlc/` if it is absent. Write `docs/sdlc/concept.md` with the
template in [ARTIFACTS.md](../status/ARTIFACTS.md).

Rules for the prose:

- Under 400 words in total.
- Plain sentences. No marketing.
- Every open question is answerable. Delete a question when you answer it.

Then create or update `docs/sdlc/state.md` with the template in
[STATE-FORMAT.md](../status/STATE-FORMAT.md). Set `concept` to `in progress`.

## The gate

Summarise the document in five bullets or fewer. Give the path. Do not paste
the document into the chat.

Then end the message with this line, and nothing after it:

> **Approve `concept`?** Reply `yes`, or tell me what to change.

No `Next` block follows the question. See
[ASKING.md](../status/ASKING.md).

On approval, set `concept` to `approved <today's date>` in `state.md`.

On a change request, edit the document and ask the same question again.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). After
approval it reads:

```
Next
  → /sdlc:explore          poke holes in this first
    /sdlc:architect        go straight to the stack
```

Recommend `/sdlc:explore` when the idea is new or the open questions are still
open. Recommend `/sdlc:architect` when the user has built this kind of thing
before and the concept is tight.

## Existing repos

The chain works on a repo that already has code. Two differences:

- The problem section describes what is wrong with the current behaviour.
- `## Out of scope` names the parts of the app this change does not touch.

The document still covers one idea. If the user has three ideas, ask which one
comes first and write that one. Note the others under `## Open questions`.

## What this skill does not do

- It does not choose a stack. That is `/sdlc:architect`.
- It does not gap-hunt or red-team. That is `/sdlc:explore`.
- It does not write code, tests, or a task list.
- It does not read source code. It reads documents.
- It does not commit.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask: one question, last line,
  nothing after it.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `concept.md` template.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
