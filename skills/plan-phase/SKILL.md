---
name: plan-phase
description: Stage 5 of the SDLC chain. Turns one phase into epics and a test plan. Drafts docs/sdlc/phase-NN-<slug>/ (phase.md with numbered done criteria, test-plan.md mapping every criterion to a test, and one epic file per vertical slice with Given/When/Then acceptance criteria) on plan/phase-NN-<slug> — using a git worktree when phases.md is committed — and commits and merges the approved plan onto the default branch. Flags any done criterion that no test covers. Use when the user says "plan phase 2", "break this phase into epics", "what are the epics", "write the test plan", or names a phase to plan. Do NOT use to decide the phase order — that is stage 4, /sdlc:phases.
argument-hint: "[phase number or slug — 01, or 01-ingest]"
---

# Plan a phase: epics and the tests that prove them

**Phase to plan:** $ARGUMENTS

## Preconditions

Read `docs/sdlc/state.md`. The `phases` stage must be `approved`. If it is not,
name the missing stage and stop.

Read `docs/sdlc/phases.md` and `docs/sdlc/architecture.md` in full. Read
`docs/sdlc/concept.md` for the problem it solves.

Resolve the phase from `$ARGUMENTS`. Accept a number (`01`, `2`) or a slug
(`01-ingest`, `ingest`). If `$ARGUMENTS` is empty, take the first phase in
`state.md` that is not `done`. If the phase does not exist in `phases.md`, say
so and stop. Do not guess.

## Work on a plan branch

Draft each phase plan on `plan/phase-NN-<slug>` so unapproved files do not
pollute the main working tree or active builds.

1. **Check for a repository with commits.** Run
   `git rev-parse --is-inside-work-tree` and `git rev-parse --verify HEAD`. If
   either fails, plan directly in the working tree without a branch and skip
   `## Merge the plan` at the end.
2. **Find the default branch.** Read
   `git symbolic-ref --quiet refs/remotes/origin/HEAD`. When there is no
   remote, use `main` if it exists, else `master`.
3. **Choose worktree or in-place branch.** Check
   `git cat-file -e <default>:docs/sdlc/phases.md`:
   - **Committed on `<default>` (phase 2 onward, and phase 1 once stages 1–4
     are committed):** Always use an isolated worktree at
     `.worktrees/plan-phase-NN-<slug>` on branch `plan/phase-NN-<slug>` so the
     main working tree stays untouched for concurrent builds or planning:
     - Run `git check-ignore -q .worktrees`. When it exits non-zero, append
       `.worktrees/` to `.git/info/exclude` so `git status` in the main tree
       stays clean without modifying `.gitignore`.
     - Reuse `.worktrees/plan-phase-NN-<slug>` if it already exists. Else if
       branch `plan/phase-NN-<slug>` exists, run
       `git worktree add .worktrees/plan-phase-NN-<slug> plan/phase-NN-<slug>`.
       Else run
       `git worktree add -b plan/phase-NN-<slug> .worktrees/plan-phase-NN-<slug> <default>`.
     - Keep the shell working directory at the repository root. Read and write
       files under `.worktrees/plan-phase-NN-<slug>/...` and pass
       `-C .worktrees/plan-phase-NN-<slug>` to git commands. Never `cd` into
       the worktree directory.
   - **Not committed on `<default>` yet (stages 1–4 still uncommitted in the
     working tree):**
     - Check `git branch --show-current`. If the main working tree is on another
       branch (neither `<default>` nor `plan/phase-NN-<slug>`), stop and tell
       the user to commit the approved `docs/sdlc/` files on `<default>` before
       planning in parallel.
     - If already on `plan/phase-NN-<slug>`, stay on it. Otherwise run
       `git checkout -b plan/phase-NN-<slug>`.
4. Name `plan/phase-NN-<slug>` (and `.worktrees/plan-phase-NN-<slug>` when
   used) in your report so the user knows where the draft sits.

## Orient on the code

Read the parts of the repo this phase touches. Use the `research` subagent
(`invoke_subagent` with `TypeName: "research"`) when the surface is wide. You
need to know:

- Which files exist already, and which the phase adds.
- How this project writes tests, and the exact command that runs them.
- Which seams the tests can use.

Read `AGENTS.md`, `CLAUDE.md`, `CONTEXT.md`, and `docs/adr/` if they exist.
Follow stable repo and tooling rules in `AGENTS.md` / `CLAUDE.md`, and match
the domain glossary terms in `CONTEXT.md`.

## Step 1 — Done criteria

Turn the phase goal into a numbered list of observable outcomes. Each one gets
an ID: `DC-1`, `DC-2`.

A good done criterion:

- Describes what a person can observe, not what the code contains.
- Is true or false. No "works well".
- Needs no other criterion to make sense.

Bad: "The parser is robust." Good: "DC-3 — A file with a malformed row loads,
and the malformed row appears in the error list."

Five to ten criteria per phase is normal.

## Step 2 — Slice into epics

An epic is one vertical slice. Three tests, and all three must pass:

1. It changes a few files, not a layer.
2. It delivers one clear behaviour.
3. Tests can prove it.

Half a day to two days of work each. Three to six epics per phase is normal.

Rules:

- **Epic 1 is the thinnest end-to-end line** through the phase.
- **No horizontal epics.** "Add the models" is not an epic.
- **Separate distinct toolchains and asset-authoring.** Keep work that uses a
  different toolchain or pipeline (such as procedural 3D scenes, asset
  authoring, or navmesh baking) in separate epics from core domain or UI logic.
- **Do not over-split tightly coupled small slices.** Avoid breaking small,
  tightly coupled slices across separate epics just to keep epics tiny — in
  `/sdlc:loop-phase`, every epic boundary pays a cold-start and multi-reviewer
  tax.
- Order epics so each one runs on top of the last.
- Say which epics are independent. Independent epics can run in any order.

Every done criterion belongs to at least one epic. Every epic covers at least
one done criterion. Say which, in both files.

## Step 3 — Acceptance criteria per epic

Write each acceptance criterion in Given/When/Then form, numbered `AC-1`,
`AC-2`. Three to six per epic.

Each acceptance criterion names a state, an action, and a result. A criterion
you cannot write as a test is not a criterion. Rewrite it or drop it.

## Step 4 — The test plan

Build the coverage table. One row per done criterion.

Mark each row `automated` or `manual`. Prefer automated. Use `manual` only when
no headless test can reach it — a visual layout, a real payment, a device
feature. A manual row needs the exact steps a person follows.

**Then hunt the gaps.** Any done criterion with no row is a gap. List every gap
under `## Gaps` with the reason. An empty gap list is the goal, not the
default. Do not invent a test to close a gap on paper.

Use the highest seam that still proves the behaviour. A test through the HTTP
route beats four tests on the functions behind it.

## Step 5 — Write the files

When working in `.worktrees/plan-phase-NN-<slug>/`, write all phase files and
update `state.md` inside `.worktrees/plan-phase-NN-<slug>/`.

Create `docs/sdlc/phase-NN-<slug>/`. Write, with the templates in
[ARTIFACTS.md](../status/ARTIFACTS.md):

- `phase.md`
- `test-plan.md`
- One `epic-NN-<slug>.md` per epic. Write only the spec block. Stage 6 and
  stage 7 append to the same file.

Then update `docs/sdlc/state.md`:

- Set the phase to `in progress`.
- Add every epic under it with status `pending`.
- Keep one blank line between phases per
  [STATE-FORMAT.md](../status/STATE-FORMAT.md).

## The gate

Report in this order, and keep it short:

1. The done criteria, one line each.
2. The epics, in build order, one line each.
3. The gaps. If there are none, say "No gaps."

Give the folder path, and name the branch (and worktree path, when used). Do not
paste the files into the chat.

**When the gap list is not empty:** Do not ask to approve yet. Name the first
uncovered done criterion, then end the message with this line and nothing after
it:

> **How do we prove `DC-N`?** Name a test, or reply `manual` to record it as a
> manual step.

Update `test-plan.md` (and the owning epic file, if automated) on the plan
branch for each gap, one question per message, until no unresolved gaps remain.

**When no unresolved gaps remain (on a plan branch):** End the message with
these two blocks and nothing after them:

> WARNING: Approval commits the plan docs (`docs/sdlc/phase-NN-<slug>/` and
> `state.md`) and merges `plan/phase-NN-<slug>` into `<default>` locally.
>
> **Approve and merge the `phase NN` plan?** Reply `yes` to commit and merge the
> plan docs, or tell me what to change.

**When there is no git commit yet (no plan branch):** End the message with this
line, and nothing after it:

> **Approve the `phase NN` plan?** Reply `yes`, or tell me what to change.

Do not call the next-steps menu in any question or gate turn. See
[ASKING.md](../status/ASKING.md).

## Merge the plan

Run this section only after a clear `yes` when working on
`plan/phase-NN-<slug>`. If there is no git repository or `HEAD` commit, skip to
`## Close`.

1. **Refresh `state.md` if `<default>` moved (worktree mode).** Check
   `git -C .worktrees/plan-phase-NN-<slug> merge-base --is-ancestor <default> HEAD`.
   If it exits non-zero (another epic landed on `<default>` while planning):
   - `git -C .worktrees/plan-phase-NN-<slug> checkout -- docs/sdlc/state.md`
   - `git -C .worktrees/plan-phase-NN-<slug> rebase <default>`
   - Re-apply this phase's `in progress` status, `pending` epic lines, and
     `Updated:` date to `.worktrees/plan-phase-NN-<slug>/docs/sdlc/state.md`.
2. **Stage and commit.** Add named paths, never `.`:
   - **Worktree mode:**
     `git -C .worktrees/plan-phase-NN-<slug> add docs/sdlc/phase-NN-<slug> docs/sdlc/state.md`
   - **In-place mode** (when stage 1–4 docs were not yet committed): stage
     `docs/sdlc/phase-NN-<slug>`, `docs/sdlc/state.md`, and any approved stage
     1–4 files under `docs/sdlc/` (`concept.md`, `architecture.md`, `design.md`,
     `phases.md`) and `docs/adr/` that are untracked or modified. Do not stage
     unrelated files.
   - Commit with Simplified Technical English, subject under 72 characters:
     ```
     docs: plan phase NN — <title>

     Add phase.md, test-plan.md, and N epic specs for phase NN.
     ```
     **Never add a `Co-Authored-By` trailer.** Never add an AI attribution
     footer of any kind.
3. **Merge into `<default>` and clean up:**
   - **In-place mode:** Run `git checkout <default>`,
     `git merge --no-ff --no-edit plan/phase-NN-<slug>`, and
     `git branch -d plan/phase-NN-<slug>`.
   - **Worktree mode:** Check `git branch --show-current` in the main working
     tree:
     - **Main tree is not on `<default>`** (for example, another agent is on
       `epic-NN-...`): Run
       `git -C .worktrees/plan-phase-NN-<slug> checkout <default>`,
       `git -C .worktrees/plan-phase-NN-<slug> merge --no-ff --no-edit plan/phase-NN-<slug>`,
       and `git -C .worktrees/plan-phase-NN-<slug> branch -d plan/phase-NN-<slug>`,
       then from the repo root
       `git worktree remove .worktrees/plan-phase-NN-<slug>`.
     - **Main tree is on `<default>`:** First run
       `git worktree remove .worktrees/plan-phase-NN-<slug>`. Then check
       `git status --short`: if `docs/sdlc/state.md` has no uncommitted changes
       in the main tree, run
       `git merge --no-ff --no-edit plan/phase-NN-<slug>` and
       `git branch -d plan/phase-NN-<slug>`. Otherwise leave
       `plan/phase-NN-<slug>` intact, report that the plan is committed on
       `plan/phase-NN-<slug>`, and give
       `git merge --no-ff plan/phase-NN-<slug>` for once `<default>` is clean.
   - If `.worktrees` exists and is empty, remove the empty directory
     (`rmdir .worktrees`).
   - On any merge conflict: stop, report the conflicting files, and do not
     resolve it yourself.

## Close

Invoke the interactive next-step menu from
[NEXT-STEPS.md](../status/NEXT-STEPS.md):

- `(Recommended) /sdlc:build-epic 01 — build the tracer bullet`
- `/sdlc:loop-phase 01 — build and land phase`
- `/sdlc:phases — phase is too big`

Name the epic number. Recommend the first epic that is not `reviewed`. Omit
`/sdlc:phases` when the slice sizes look right.

When two epics are independent, include both in the options and note that either
can go first.

When the plan merged into `<default>`, omit the `Uncommitted:` reminder because
the plan is committed. Print `Uncommitted:` right before calling the tool only
when there was no git repository or `HEAD` commit (or when a dirty main tree
kept the plan on `plan/phase-NN-<slug>`).

## What this skill does not do

- It does not write code or tests. It names them.
- It does not run the test suite.
- It does not change `phases.md` or `architecture.md`. If the phase does not
  fit the architecture, say so and stop.
- It does not plan more than one phase at a time.
- It does not push to a remote. Approval commits and merges locally.
- It does not force push or rewrite history.

## Reference

- [ASKING.md](../status/ASKING.md) — plain-text questions and gates vs.
  interactive menus.
- [NEXT-STEPS.md](../status/NEXT-STEPS.md) — the interactive closing menu.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `phase.md`, `test-plan.md`, and
  epic file templates.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
