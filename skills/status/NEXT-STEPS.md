# How every skill closes

Every non-question turn in the chain closes with an interactive next-step menu.
The user never has to work out or type what runs next.

## Presenting next steps

At the end of a turn that does not ask a plain-text question, invoke the host's
multiple-choice question tool:

- **Antigravity (`ask_question`):** Call `ask_question` with one question
  (`"What should run next?"`), `is_multi_select: false`, and `options` as 2–4
  strings. Put the recommended step first, prefixed with `(Recommended)`:
  - `"(Recommended) /sdlc:architect — decide stack and shape"`
  - `"/sdlc:explore — deepen the concept first"`
  - `"/sdlc:status — see the whole project"`
- **Claude Code (`AskUserQuestion`):** Call `AskUserQuestion` with one question
  (`header: "Next"`, `question: "What should run next?"`, `multiSelect: false`)
  and 2–4 `options` objects. Put the recommended step first with `(Recommended)`
  on `label` and the short reason in `description`:
  - `label: "/sdlc:architect (Recommended)"`, `description: "decide stack and shape"`
  - `label: "/sdlc:explore"`, `description: "deepen the concept first"`
  - `label: "/sdlc:status"`, `description: "see the whole project"`
- **Plain-text fallback (no interactive question tool in the harness):** Only
  when neither `ask_question` nor `AskUserQuestion` is available, end the
  message with `**What next?**` followed by a bulleted list of the same 2–4
  options, with `(Recommended)` on the first item.

Rules:

- **Exactly one recommendation, always first.** Mark only the first option
  `(Recommended)`.
- **Two to four options.** Never fewer than two and never more than four. Never
  add a manual `"Other"` option — both UIs provide `"Other"` automatically.
- **Full command with its argument.** Write `/sdlc:build-epic 03`, not
  `/sdlc:build-epic`.
- **Portable command names.** Always write `/sdlc:<skill>` in option labels so
  Claude Code plugin namespacing works. In harnesses where skills load without a
  plugin prefix (such as Antigravity), `/sdlc:<skill>` maps directly to the
  `<skill>` skill (`skills/<skill>/SKILL.md`).
- **Four words or fewer for each reason.** Say what the option does for the
  user, not what the skill is called.
- **Give the work summary and situation before calling the tool.** Because
  `ask_question` / `AskUserQuestion` pauses the turn immediately (preventing any
  post-tool wrap-up), always emit the stage's work summary, report, open manual
  checks or warnings, and any `Uncommitted:` line in your assistant message text
  right before calling the tool. Never invoke the tool with an empty message or
  bare bookkeeping alone, and print nothing redundant after it.
- **Never call the next-steps menu in a turn that asks a plain-text question.**
  Interview questions, destructive confirmations, and `Approve <stage>?` gates
  end with the bold plain-text question alone. See [ASKING.md](ASKING.md). Call
  the next-steps menu only after the user answers.

## Acting on the choice

The interactive tool blocks until the user responds, then returns their choice:

- **A skill option (for example `/sdlc:architect` or `/sdlc:build-epic 03`):**
  Proceed directly into that skill with its argument — invoke it via the host's
  `Skill` tool in Claude Code, or read `skills/<skill>/SKILL.md` and execute it
  directly in Antigravity. Do not ask the user to type the command.
- **A pause or stop option (for example `Pause before review`, `Pause here`, or
  `Done`):** Print any required closing warning (such as `build-epic`'s
  unreviewed-code warning), run no further stage, and end the turn.
- **A custom `"Other"` reply:** Honor what the user wrote. If they asked a
  question, answer it and stop without advancing the chain. If they asked for
  edits, make them and re-ask or close as appropriate. If they named another
  skill or command, run that instead.

## Choosing the recommendation

Read `docs/sdlc/state.md`, then take the first row that matches.

| State | Recommend |
|---|---|
| An epic is `blocked` | `/sdlc:build-epic <epic>` or `/sdlc:review <epic>` to unblock |
| An epic is `reviewed` | `/sdlc:push-epic <epic>` |
| An epic is `built` | `/sdlc:review <epic>` |
| An epic is `building` | `/sdlc:build-epic <epic>` |
| The current phase has a `pending` epic | `/sdlc:build-epic <next epic>` |
| Every epic in the phase is `merged` | `/sdlc:plan-phase <next phase>` |
| `phases` is approved, no phase planned | `/sdlc:plan-phase 01` |
| `architecture` is approved | `/sdlc:phases` |
| `design` is in progress | finish `/sdlc:design` |
| `concept` is approved | `/sdlc:architect` |
| `concept` is in progress | finish `/sdlc:concept` |
| Every phase is `done` | `Done — finish the run` |

A stage the user has not approved always outranks a later stage. Never
recommend skipping a gate.

When an epic is `built`, Stage 6 closes directly with this interactive menu:
`/sdlc:review <epic>` is `(Recommended)` first, paired with a `Pause before
review` option that prints the skipped-review warning if chosen.

## The alternatives

Below the `(Recommended)` option, include 1–3 branches that are genuinely open.
Pick from these; never list all of them:

- **The optional stage.** `/sdlc:explore` after `/sdlc:concept`, and
  `/sdlc:design` after `/sdlc:architect` (only when the project has a UI).
- **The way back.** When the stage exposed a flaw in an earlier document, offer
  that earlier skill and name the flaw in <= 4 words.
- **The wider view.** `/sdlc:status` when the user has been inside one epic or
  needs a neutral second option.
- **The parallel epic.** When `plan-phase` marked two epics independent, offer
  either one.
- **The open issue.** `/sdlc:issue` when `issues.md` holds an open issue that
  the recommended step does not cover. Name the issue ID.
- **The whole phase.** `/sdlc:loop-phase <phase>` when a phase is about to
  start, or when epics remain in the current phase.
- **The rebuild.** `/sdlc:respec <scope>` when a review failed twice on one
  epic, or when build notes show the same deviation repeatedly.
- **A clean pause.** `Pause here — commit files first` when uncommitted files
  should be committed manually before continuing.

`/sdlc:respec` never appears in the recommendation table above. Offer it only
when the evidence names a shape problem.

Do not offer a step whose preconditions are not met. Do not offer
`/sdlc:build-epic` for an epic that no phase plan covers.

## When the chain stops

Four cases stop or redirect the chain while still presenting 2–4 valid options:

- **The idea died.** `/sdlc:explore` killed it. Say in the message that the
  exploration notes are worth keeping, then offer
  `(Recommended) /sdlc:concept — start a fresh idea` and
  `Stop here — keep exploration notes`.
- **The stage is blocked on the user.** A spike must run, a test failed three
  times, or a decision needs a person. Explain what is needed and which skill
  resumes in the message text, then offer `(Recommended) Stop here — run the
  spike first` (or `Stop here — inspect failure first`) alongside the skill that
  resumes or `/sdlc:status — see the whole project`.
- **A precondition is missing.** When a skill stops because `state.md` or an
  earlier approved stage is missing, name what is missing and offer
  `(Recommended) /sdlc:<missing-stage> — <short reason>` alongside
  `Stop here — do nothing now`.
- **Every phase is done.** When a phase finishes and another remains, recommend
  `/sdlc:plan-phase <next phase>` (or `/sdlc:loop-phase <next phase>`). When no
  phase remains, say the project is done in the message and offer
  `(Recommended) Done — finish the run` and `/sdlc:status — view final record`.

## Commit reminder

`/sdlc:plan-phase` commits and merges an approved phase plan locally onto the
default branch, and `/sdlc:push-epic` commits, pushes, and merges a reviewed
epic branch. Stages 1 to 4 and stages 6 to 7 leave changes in the working tree.

When a skill left uncommitted files in the repo, print one line at the end of
the message right before calling the interactive question tool:

```
Uncommitted: 4 files changed. Commit before you continue.
```

For a code change on an epic branch, recommend `/sdlc:push-epic` instead of a
manual commit once the epic is `reviewed`. Stages 1 to 4 change documents, which
the user commits (or commits automatically if they later land `plan-phase` in
in-place mode).
