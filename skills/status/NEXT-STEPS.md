# How every skill closes

Every skill in the chain ends with the same block. The user never has to work
out what runs next.

## The block

```
Next
  → /sdlc:architect        decide the stack and the shape
    /sdlc:explore          deepen the concept first
    /sdlc:status           see the whole project
```

Rules:

- The arrow marks the recommendation. There is always exactly one arrow.
- Two to four lines. Never more.
- Give the full command, with its argument. `/sdlc:build-epic 03`, not
  `/sdlc:build-epic`.
- Each line gets a short reason, four words or fewer. Say what the skill does
  for the user, not what it is called.
- Put the block last. Nothing follows it.
- **Never print the block in a message that asks a question.** A question ends
  its message, and the block would bury it. See [ASKING.md](ASKING.md). Print
  the block in the next message, after the user answers.
- So a gate message ends with the gate question and no block. The block comes
  after the approval, or after the user says what to change.

## Choosing the recommendation

Read `docs/sdlc/state.md`, then take the first row that matches.

| State | Recommend |
|---|---|
| An epic is `blocked` | `/sdlc:review <epic>` findings first |
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
| Every phase is `done` | say the project is done |

A stage the user has not approved always outranks a later stage. Never
recommend skipping a gate.

## The alternatives

Under the recommendation, offer the branches that are genuinely open. Pick from
these. Do not list all of them.

- **The optional stage.** `/sdlc:explore` after `/sdlc:concept`, and
  `/sdlc:design` after `/sdlc:architect`. Say it is optional. Offer
  `/sdlc:design` only when the project has a user interface.
- **The way back.** When the stage exposed a problem in an earlier document,
  offer that earlier skill. Name the problem in the reason.
- **The wider view.** `/sdlc:status` when the user has been in one epic for a
  while.
- **The parallel epic.** When `plan-phase` marked two epics independent, offer
  either one.
- **The open issue.** `/sdlc:issue` when `issues.md` holds an issue that the
  recommended step does not cover. Name the issue ID in the reason.
- **The whole phase.** `/sdlc:loop-phase <phase>` when a phase is about to
  start, or when epics are left in the current phase. It asks once, then runs
  stages 5 to 8 to the end of the phase.

Do not offer a step that its own preconditions block. Do not offer
`/sdlc:build-epic` for an epic that no phase plan covers.

## When the chain stops

Three cases end the chain instead of advancing it:

- **The idea died.** `/sdlc:explore` killed it. Recommend
  `/sdlc:concept` to start again, and say the exploration is worth keeping.
- **The stage is blocked on the user.** A spike must run, or a decision needs
  a person. Say what you need, then recommend the skill that resumes.
- **The phase is done.** Recommend the next phase. When no phase is left,
  say the project is done and recommend `/sdlc:status` for the record.

## Commit reminder

Only `/sdlc:push-epic` commits. Every other skill leaves the changes in the
working tree.

When a skill changed files in the repo, put one line above the `Next` block:

```
Uncommitted: 4 files changed. Commit before you continue.
```

For a code change on an epic branch, recommend `/sdlc:push-epic` instead of a
manual commit, once the epic is `reviewed`. The document stages change
documents, which the user commits.
