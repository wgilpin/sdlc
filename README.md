# Personal SDLC

One chain of skills from a half-formed idea to reviewed code. Each stage writes
a file to `docs/sdlc/` in the target repo. A state file records where you are,
so a fresh session continues from disk alone.

The chain replaces spec-kit and the `software-factory` skill. It depends on
neither.

## The chain

```
/sdlc:concept  →  /sdlc:explore  →  /sdlc:architect  →  /sdlc:phases
                                          ↓                  ↑
                                    /sdlc:design ────────────┘
                                                             ↓
/sdlc:push-epic ← /sdlc:review ← /sdlc:build-epic ← /sdlc:plan-phase
       ↓                                                     ↑
   next epic ────────────────────────────────────────────────┘
```

| Skill | Stage | What it does |
|---|---|---|
| `/sdlc:concept` | 1 | Six questions, one at a time. Writes a short `concept.md`. |
| `/sdlc:explore` | 2 | Deepens the same `concept.md`. Gap-hunts and red-teams it. |
| `/sdlc:architect` | 3 | A working session on how to build it. Writes `architecture.md`. |
| `/sdlc:design` | 3b | Optional. Settles the UI. Writes `design.md`, and can drive Stitch. |
| `/sdlc:phases` | 4 | Breaks the work into shippable phases, with you. |
| `/sdlc:plan-phase` | 5 | Turns one phase into epics and a test plan. |
| `/sdlc:build-epic` | 6 | Runs one epic red-green-refactor to green, on its own branch. |
| `/sdlc:review` | 7 | Checks the diff against the epic, concept, architecture, and security rules. |
| `/sdlc:push-epic` | 8 | Commits, pushes, merges into the default branch, pulls. |
| `/sdlc:loop-phase` | 5–8 | Runs a whole phase in one go. One gate, then no stops. |
| `/sdlc:issue` | — | Routes one issue: fix it now, or into the epic or phase that owns it. |
| `/sdlc:respec` | — | Writes a fresh spec to rebuild a fix, an epic, a phase, or the app. |
| `/sdlc:status` | — | Says where you are and what runs next. |
| `/sdlc:help` | — | Describes the chain, the rules, and one stage on request. |

Stages 5 to 8 repeat for every epic. Stages 4 to 8 repeat for every phase.

Stages 2 and 3b are optional. Stage 3b applies only to a project with a user
interface. It needs no login of its own — the Stitch step runs through the
Stitch MCP, and the Claude Design push at the end is a further offer you can
decline.

## Version control

`/sdlc:build-epic` refuses to write code in a directory with no git
repository. It offers `git init` and waits for an answer.

`/sdlc:plan-phase` drafts on `plan/phase-NN-<slug>` (inside `.worktrees/` once
`phases.md` is committed, so you can plan the next phase while an epic builds)
and merges the approved plan locally into the default branch.

Each epic gets its own branch, `epic-NN-<slug>`, cut from an up-to-date default
branch. `/sdlc:push-epic` lands it and returns you to the default branch.

The default branch name is detected, never assumed. `main` and `master` both
work.

## Install

From a local clone:

```bash
claude plugin marketplace add /Users/will/projects/sdlc
```

From GitHub:

```bash
claude plugin marketplace add wgilpin/sdlc
```

Then install the plugin and restart the session:

```bash
claude plugin install sdlc@will
```

In Antigravity, load the `skills/` folders directly — every `/sdlc:<skill>`
command in the chain maps to the `<skill>` skill (`concept`, `architect`,
`build-epic`, and so on) without the prefix.

## What it writes

```
docs/sdlc/
  state.md                  where you are in the chain
  concept.md                stage 1, deepened by stage 2
  architecture.md           stage 3, records the stack
  design.md                 stage 3b, tokens and surfaces, UI projects only
  design-kit/               stage 3b, component kit, Claude Design push only
  phases.md                 stage 4, the ordered phase list
  issues.md                 /sdlc:issue, the record of every issue raised
  phase-01-<slug>/
    phase.md                stage 5: goal, scope, done criteria
    test-plan.md            stage 5: how the phase gets proven
    epic-01-<slug>.md       stage 5 spec, stage 6 build note, stage 7 review
docs/adr/NNNN-<slug>.md     decisions that are hard to reverse
docs/sdlc-v2/               /sdlc:respec, a spec to rebuild from scratch
AGENTS.md / CLAUDE.md       repo-wide invariants promoted when an epic lands
```

One file per epic. That file holds the spec, the build note, and the review, in
that order. The acceptance criteria that `/sdlc:review` checks sit at the top of
the same file it writes to.

## How the stages connect

Every stage skill does the same three things:

1. It reads `docs/sdlc/state.md`.
2. It checks its preconditions. If an earlier stage is not approved, it names
   the missing stage and stops.
3. It does the work, then updates `state.md` in the same turn.

(`/sdlc:status` and `/sdlc:respec` read `state.md` without changing it;
`/sdlc:help` reads no project files.)

Stages 1 to 4 end with **Approve `<stage>`?** Reply `yes`, or tell me what to
change. Stage 5 ends with **Approve and merge the `phase NN` plan?** Reply `yes`
to commit and merge the plan docs, or tell me what to change (or **Approve the
`phase NN` plan?** before the first commit). Nothing advances until you answer.

Stage 6 is different. It confirms the task list once, then runs to green
without a stop.

## Running a whole phase

`/sdlc:loop-phase 02` does stages 5 to 8 for one phase in a single run. It
plans the phase, then builds, reviews, and lands every epic in order. Each epic
builds in a fresh subagent, so a long phase keeps its detail and the reviewer is
not the builder.

It asks three things before it starts — a repository, a clean working tree, a
green suite — and then one gate that shows the epics and warns that the run
commits, pushes, and merges. That yes is the permission for every git write in
the run (including promoting confirmed repo-wide invariants from build notes
into `AGENTS.md` or `CLAUDE.md`). It covers that phase only.

After the yes it takes its own recommendation at every step. It stops only when
a rule forces it: a stuck test, a review that fails twice on one epic, a merge
conflict, a red suite, a `docker` or migration command, or a contradiction with
the architecture. It never deletes a branch and never softens a verdict to keep
going.

Run `/sdlc:loop-phase NN` again to pick up the epics that are left.

## Starting a part again

Sometimes the problem is not a bug. The approach fought itself, and the fix is a
rebuild. `/sdlc:respec <scope>` writes the spec for that rebuild.

It reads the artifacts, the build notes, the review findings, the code, and the
session, then writes a fresh spec that throws the old implementation away. Scope
is one fix, one epic, one phase, or the whole app. The architecture inside the
scope goes. The architecture outside it stands, and each new file names it.

Every lesson in the new spec carries its evidence: a review finding, an issue
ID, a build-note deviation, or a file. That evidence is the reason a respec
beats a fresh run of the chain.

The output is a parallel tree, `docs/sdlc-v2/`, in the same layout as
`docs/sdlc/`. Nothing under `docs/sdlc/` changes, and the state file is
untouched. The old tree stays as history. `/sdlc:respec` writes the spec and
stops — it never starts the rebuild.

## How the chain asks and closes

Every turn ends in one of two ways, so you never wonder whose turn it is:

**Plain-text questions and gates end on a bold last line.** Interview
questions, `Approve <stage>?` gates, and destructive confirmations sit on the
last line of the message, with the question in bold followed by the answer you
can give. Nothing follows that line — no summary, no closing note, and no
next-step menu. One question per message; a skill with three offers asks across
three messages. See `skills/status/ASKING.md`.

**Multi-option choices and non-question turns use an interactive menu.** For
3+ option routing choices (such as `respec` scope selection or dirty-tree
triage) and at the close of every turn that asks no plain-text question, the
skill invokes the host's multiple-choice question tool (`ask_question` in
Antigravity / `AskUserQuestion` in Claude Code, with a plain-text fallback if
neither is available):

- `(Recommended) /sdlc:build-epic 06 — build the last epic`
- `/sdlc:status — see the whole project`

The recommended step comes first, with 2–4 valid options and a short reason
(`<= 4` words) on each. Selecting an option launches that stage immediately. A
skill never offers a step whose preconditions are not met. See
`skills/status/NEXT-STEPS.md`.

## What the chain never does

- Only `/sdlc:plan-phase`, `/sdlc:push-epic`, and `/sdlc:loop-phase` run git
  write commands, and each asks at its gate first. Stages 1 to 4 and 6 to 7
  leave changes in the working tree.
- It never force pushes and never rewrites history.
- It never lands an epic that failed review.
- It never stages `.env*` files, private keys, local databases, or hard-coded
  secrets.
- It never runs `docker` or migration commands without a question first.
- It never writes code before stage 6.
- It never deletes a document. `/sdlc:respec` writes a new tree beside the old
  one and leaves the old one intact.

## Make it yours

The skills are plain markdown. Read them, disagree with them, edit them. Two
places are worth an early edit:

- `skills/build-epic/SKILL.md` finds your test command from the project. If
  your project is unusual, name the command in `architecture.md` under
  `## Stack`.
- `skills/phases/SKILL.md` assumes a phase is one to three weeks of work.
  Change that number if your phases are smaller.

## Licence

MIT. See [LICENSE](LICENSE).
