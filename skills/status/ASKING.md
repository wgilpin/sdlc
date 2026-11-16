# How every skill asks

The user must never wonder whether it is their turn. Every turn closes in
exactly one of two ways:

1. **A bold plain-text question on the last line** — for interview questions,
   stage approval gates, and binary yes/no or destructive confirmations.
2. **The host's interactive multiple-choice question tool** (`ask_question` in
   Antigravity / `AskUserQuestion` in Claude Code) — for closing next-step menus
   ([NEXT-STEPS.md](NEXT-STEPS.md)) and genuine 3+ option routing choices.

Never mix both in the same turn.

## Plain-text questions and gates

Keep open-ended questions, `Approve <stage>?` gates, and binary yes/no prompts
in plain text:

- Antigravity's `ask_question` forbids trivial yes/no tool calls.
- Document gates take either `yes` or free-form revision notes, which are faster
  to type directly than routing through an `"Other"` box.
- Destructive and git-write confirmations keep an explicit plain-text barrier.

One rule governs every plain-text ask:

**The question is the last line of the message. Nothing follows it.**

No summary, no closing note, no reassurance, and never the interactive next-step
menu. When anything follows the question, the message reads as a report and
buries the ask. Present the next-step menu only in the following turn, after the
user answers.

### The shape

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

### Naming the answer

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

### The hazard line

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

## Interactive multi-option choices

Use `ask_question` (Antigravity) or `AskUserQuestion` (Claude Code) when the
user chooses between 3+ distinct actions (such as `concept`'s `deepen` /
`replace` / `move on`, `respec`'s scope picker, or dirty-tree triage) and at the
close of every non-question turn ([NEXT-STEPS.md](NEXT-STEPS.md)):

- Put all context, file lists, `WARNING:` lines, and `Uncommitted:` reminders in
  the message text **before** calling the tool.
- Pass 2–4 options, single-select, with `(Recommended)` on the first option when
  one path is preferred. Never add a manual `"Other"` option.
- Fall back to a bold plain-text question or bulleted `**What next?**` list only
  when neither interactive tool is available in the harness.

## One question per message

Ask one thing. Wait. Ask the next thing.

A skill with three offers in it — Stitch, a component kit, the gate — asks
three times, in three messages. Never bundle them into a list of questions and
never accept one answer for two questions.

## When you are not asking

A message that asks no plain-text question must not end with vague prose like
"let me know" or "shall I continue". Close with the interactive next-step menu
from [NEXT-STEPS.md](NEXT-STEPS.md) instead so the user can launch the next
stage with one click.

## Stage 6 and autonomous loops

Stage 6 (`build-epic`) confirms the task list once, then runs to green. Inside
that run it does not ask. A blocked build stops and reports; it does not open a
negotiation. The exceptions are the user's own rules: a `docker` command, a
migration, or a database reset always asks in plain text, even mid-build.

On green, Stage 6 closes with the interactive next-step menu from
[NEXT-STEPS.md](NEXT-STEPS.md), putting `/sdlc:review <epic>` first as
`(Recommended)` alongside a `Pause before review` option that prints the
skipped-review warning if chosen. Unreviewed code still cannot walk into stage
8. Inside `/sdlc:loop-phase`, Stage 6 asks nothing because the loop gate covers
both the build and the review.
