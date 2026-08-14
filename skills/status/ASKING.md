# How every skill asks

The user must never wonder whether it is their turn. One rule does most of the
work:

**The question is the last line of the message. Nothing follows it.**

No `Next` block, no summary, no note, no reassurance. When you print anything
after the question, the message reads as a report, and the user does not know
that you stopped and waited.

## The shape

```
<Everything the user needs to decide. Lists, diffs, paths.>

<Hazard line, when the action is hard to undo.>

**<The question>?** <How to answer.>
```

Three rules for the last line:

1. It ends with a question mark, then the answer options. Nothing else.
2. It is bold, so it survives a long message.
3. It names what a yes does. `Land epic-02?` is enough only when the message
   above it said what landing means.

## Naming the answer

Never stop on a bare question. Say what to type.

| Kind of ask | The last line |
|---|---|
| A gate on a document | **Approve `concept`?** Reply `yes`, or tell me what to change. |
| A destructive action | **Land `epic-02`?** Reply `yes` to run all seven commands, or `no` to stop. |
| A choice of two | **Which one — A or B?** Name one, or ask for a third. |
| A yes or no with a default | **Generate the screens in Stitch?** Reply `yes`, or `skip` to move on. |
| Something you need from the user | **What is the test command?** Give the exact line. |

`yes` means yes. A question in reply is not a yes. A "sounds good" is a yes. If
the reply is ambiguous, ask again. Do not act on a maybe.

## The hazard line

Put the hazard **before** the question, never after it. A hazard after the
question buries the thing the user must answer.

Wrong:

```
Land epic-02? This commits, pushes, merges into main, and cannot be undone
with one command.
```

Right:

```
WARNING: This commits, pushes, and merges into `main`. One command does not
undo it.

**Land `epic-02`?** Reply `yes` to run the sequence, or `no` to stop.
```

Give one hazard, not a list. Name the worst outcome only.

## One question per message

Ask one thing. Wait. Ask the next thing.

A skill with three offers in it — Stitch, a component kit, the gate — asks
three times, in three messages. Never bundle them into a list of questions and
never accept one answer for two questions.

## When you are not asking

A message that does not end with a question must not look like one. Do not
close a report with "let me know" or "shall I continue". Print the `Next` block
from [NEXT-STEPS.md](NEXT-STEPS.md) instead. The arrow tells the user what to
run, and the absence of a question tells them the turn is theirs to spend how
they like.

## The one exception

Stage 6 confirms the task list once, then runs to green. Inside that run it
does not ask. A blocked build stops and reports; it does not open a
negotiation. The exceptions are the user's own rules: a `docker` command, a
migration, or a database reset always asks, even mid-build.
