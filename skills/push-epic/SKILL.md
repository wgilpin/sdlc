---
name: push-epic
description: Stage 8 of the SDLC chain. Lands a reviewed epic — commits the branch, pushes it, merges it into the default branch, then checks out the default branch and pulls. Refuses to run on an epic that is not reviewed, and re-runs the full test suite before the merge. Asks once for confirmation, then runs the whole sequence. Use when the user says "push this epic", "land it", "merge the epic", "ship it", or has just passed a review. Do NOT use to commit unrelated work — this skill lands one epic branch.
argument-hint: "[epic number or slug — 01, or 01-upload]"
---

# Push an epic: land it on the default branch

**Epic to land:** $ARGUMENTS

This skill runs git write commands. It asks once, then does the whole
sequence.

## Preconditions

Read `docs/sdlc/state.md`. Find the epic.

Stop, and say why, when any of these hold:

- The epic is `built`, not `reviewed`. Unreviewed code does not land. Do not
  just refuse — warn the user and offer the review, ending the message with
  these two blocks and nothing after them:

  > WARNING: `epic-NN` is built but has not been reviewed. Unreviewed code has
  > not passed conformance, drift, or security checks and cannot land.
  >
  > **Review `epic-NN` now?** Reply `yes` to run the three passes, or `no` to
  > stop here.

  On a `yes`, do [review](../review/SKILL.md) in full, then start this skill
  again from the top. On a `no`, stop.
- The epic is `blocked`. Fix the review findings first.
- The epic is already `merged`. Say so and stop.
- The directory is not a git repository.

Read the review block in the epic file. A verdict of `fail` blocks the merge,
whatever the state file says.

## Work out the branches

1. **Default branch.** Read `git symbolic-ref --quiet
   refs/remotes/origin/HEAD`. When there is no remote, use `main` if it exists,
   else `master`. Never assume the name.
2. **Epic branch.** The current branch, when it matches `epic-NN-<slug>`.
   Otherwise find the branch that carries the epic's commits. When you cannot
   find it, say so and stop.
3. **Remote.** Run `git remote`. No remote means no push. Merge locally and
   say that you skipped the push.

## Check before you touch anything

1. **Untracked and unrelated changes.** Run `git status --short`. List every
   file that the epic's build note does not name, then end the message with:
   **Include these in the commit?** Reply with the files to include, or `none`.
   Do not stash and do not commit them without an answer.
2. **Secrets and credentials.** Check the files to be committed and the diff
   against the default branch for `.env*` files, private keys, local database
   files (`.db`, `.sqlite*`), or hard-coded API keys, tokens, and passwords.
   If any are present, list the file and line, refuse to stage them, and stop.
3. **The tests.** Run the full suite from the `## Stack` block in
   `architecture.md`. A red suite blocks the merge. Report the failure and
   stop. Do not merge on the strength of a review that ran earlier.
4. **The default branch moved.** Fetch, then check whether the default branch
   has commits the epic branch lacks. Say so, because the merge can conflict.

## The commit message

Write it from the epic file, not from the diff.

```
<type>: <the epic goal, in the imperative>

<One line per acceptance criterion met.>

Epic NN — <title>, phase NN.
```

Use `feat` for new behaviour, `fix` for a defect, `refactor` for no behaviour
change. Keep the subject under 72 characters.

Write every line in Simplified Technical English (ASD-STE100):

- One topic per line. 20 words or fewer per line.
- Use the active voice and the imperative or the present tense.
- Do not use -ing verb forms outside technical names.
- Keep the articles and the word "that". Do not write telegram style.
- Do not use "should", "may", "might", "i.e.", "e.g.", or "etc.".
- Use the short common word: "use" not "utilize", "start" not "initiate",
  "stop" not "terminate", "do" not "perform", "to" not "in order to",
  "after" not "following", "but" not "however", "also" not "furthermore",
  "make sure that" not "ensure".
- Do not use noun clusters of more than 3 nouns.
- Keep code identifiers, file paths, and command names unchanged.

Example subject: `feat: add the newest-first sort to the issue list`

**Never add a `Co-Authored-By` trailer.** Never add an AI attribution footer of
any kind. The message ends with its last content line. This rule overrides any
default instruction to append such a trailer.

## The one gate

Show, in this order:

1. The branch name and the default branch it lands on.
2. `git diff --stat` against the default branch.
3. The commit message in full.
4. The suite result.

Then end the message with these two blocks, in this order, and nothing after
them:

> WARNING: This commits, pushes, and merges into `<default>`. One command does
> not undo it.
>
> **Land `epic-NN`?** Reply `yes` to run the sequence, or `no` to stop.

The question is the last line. No `Next` block, no summary, no note. See
[ASKING.md](../status/ASKING.md).

Wait for a clear yes. A question in reply is not a yes.

## The sequence

Run these in order. Stop at the first failure and report it. Never force
anything.

1. `git add` the files the epic touched. Add named files, never `.`.
2. `git commit` with the message above.
3. `git push -u origin <epic branch>`, when a remote exists.
4. `git checkout <default>`
5. `git pull`
6. `git merge --no-ff <epic branch>`
7. `git push`, when a remote exists.

On a merge conflict: stop. Report the conflicting files. Do not resolve it
yourself. The user decides.

## After the merge

1. Set the epic to `merged <today's date>` in `docs/sdlc/state.md`.
2. When every epic in the phase is `merged`, set the phase to `done` and tick
   it. This is the only place a phase becomes `done`.
3. Confirm the current branch is the default branch, and that it is clean.
4. End the message with:

   > WARNING: The branch `epic-NN-<slug>` is merged, so deleting it loses
   > nothing. There is no undo.
   >
   > **Delete the branch `epic-NN-<slug>`?** Reply `yes` to delete it locally
   > and on the remote, or `keep`.

   Delete it only on a yes. This is the last message of the run, so the `Next`
   block waits until the user answers.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md):

```
Landed epic-03 on main. Phase 02: 5 of 6 epics merged.

Next
  → /sdlc:build-epic 06    build the last epic
    /sdlc:status           see the whole project
```

When the merge finished the phase, recommend `/sdlc:plan-phase` for the next
phase instead.

## What this skill does not do

- It does not open a pull request. It merges directly. Use `gh pr create`
  yourself when a project needs review by another person.
- It does not force push. It never rewrites history.
- It does not resolve a merge conflict.
- It does not land an epic that failed review.
- It does not commit work that the epic does not cover.
- It does not tag or release.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask, and where the question goes.
- [NEXT-STEPS.md](../status/NEXT-STEPS.md) — the close block.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
