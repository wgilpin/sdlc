# Artifacts

Every file the chain writes, and the template for each. One place, so the
formats never drift apart.

## Layout

```
docs/sdlc/
  state.md                  see STATE-FORMAT.md
  concept.md                stage 1, deepened by stage 2
  architecture.md           stage 3
  design.md                 stage 3b, only when the project has a UI
  design-kit/               stage 3b, only when pushed to Claude Design
  phases.md                 stage 4
  issues.md                 /sdlc:issue, only once an issue is raised
  phase-01-<slug>/
    phase.md                stage 5
    test-plan.md            stage 5
    epic-01-<slug>.md       stage 5, then stage 6, then stage 7
    epic-02-<slug>.md
  phase-02-<slug>/
    ...
docs/adr/NNNN-<slug>.md     stage 3 and stage 6
```

## Naming

- Numbers are two digits, from `01`, in build order.
- Slugs are kebab-case, three words or fewer, from the phase or epic title.
- Numbers never change once written. A phase inserted later takes the next
  free number and moves in the ordered list in `phases.md`.
- ADR numbers are four digits, from `0001`. An ADR is never rewritten. A later
  ADR supersedes it.

## concept.md

```markdown
# <Idea title>

## Problem

<Two to four sentences. What is wrong today, and for whom.>

## Who it is for

<The user. One paragraph. Name the person, not a segment.>

## The smallest thing that proves it

<What must work for you to know the idea holds. Not the full product.>

## Out of scope

<A bullet list. What this deliberately does not do.>

## Open questions

<A bullet list. Each question is answerable. Delete a question when it is
answered and fold the answer into the sections above.>
```

Stage 2 adds three sections to the same file:

```markdown
## Decisions made while we explored

<Each decision, and the reason for it. One line each.>

## Risks

<What can make this fail. Each risk with the evidence that it is real.>

## Feasibility flags

<Anything that is hard, slow, or costs money. Each with a rough size.>
```

## architecture.md

```markdown
# Architecture — <project name>

## Goals

<What the architecture must achieve, from concept.md. Three bullets or fewer.>

## Constraints

<What limits the choice. Budget, hosting, existing code, your own rules.>

## Approaches considered

<Two or three. Each with a name, one paragraph, and its trade-off.>

## Recommended approach

<Which one, and why. Name the deciding factor.>

## Stack

<One line per fact. This block is copied into state.md verbatim.

- Language: <name and version>
- Framework: <name>
- Database: <name and version>
- Tests: <runner, and the exact command to run the full suite>
- Deploy: <where it runs>
>

## Shape of the system

<The parts, and how they talk. A short list or a small diagram. Name the
modules and the seams between them.>

## Data

<The main entities and how they relate. Field-level detail only where it
drives a decision.>

## Key decisions

<Each decision, the alternative rejected, and the reason. Link to an ADR
when one exists.>

## Risks and spikes

<What to prove before you build on it. Each spike with a question it answers
and a time box.>

## Open questions
```

## design.md

Written by stage 3b. Absent for a project with no user interface.

The file has two readers. Stage 6 builds the UI from it, and Stitch takes it as
its `DESIGN.md`. Give real values, never a placeholder.

```markdown
# Design — <project name>

**Look:** <three words the user agreed to>

## Surfaces

<One line per screen: its name, what it is for, and the one thing a person
does there.>

## Primary flow

<The path that proves the concept, screen by screen. Numbered.>

## Tokens

<Literal values. This block drives the Stitch design system.

- Colour: background, surface, text, muted text, primary, danger
- Type: family, and the scale in px or rem
- Spacing: the base unit and the steps
- Radius: the values in use
- Shadow: the values in use, or "none"
>

## Components

<One line per component: its name, its variants, and where it appears.>

## States

<Per surface, or once when they are the same: empty, loading, error, and
no-permission.>

## Responsive

<The widths that matter. Name the smallest one supported.>

## Accessibility

<Contrast target, focus style, keyboard path, and the labels that matter.>

## Where the screens live

<Stitch project id, Stitch design system id, Claude Design project id. Write
"none" for any that does not exist.>

## Out of scope
```

## phases.md

```markdown
# Phases — <project name>

<One paragraph: the arc from phase 1 to the last phase.>

## Phase 01 — <title>

**Goal:** <one sentence>

**Works after this phase:** <what you can do that you could not do before>

**Not in this phase:** <what it deliberately leaves out>

**Depends on:** <earlier phases, or "nothing">

## Phase 02 — <title>

...
```

Status does not live here. Status lives in `state.md`.

## issues.md

Written by `/sdlc:issue`. Absent until the first issue is raised.

```markdown
# Issues — <project name>

One line per issue, newest last. Status is `open`, `scheduled`, `fixed`,
or `dropped`. This file records issues. It never records stage status.

## ISS-N — <the issue in one sentence>

**Raised:** <YYYY-MM-DD>
**Status:** open
**Lands in:** <epic-NN-<slug> / phase NN / unplanned / fixed on <branch>>

<Two lines at most. What is wrong, and how you know.>
```

An issue keeps its number for good. A closed issue stays in the file with a
new status, never deleted.

## phase.md

```markdown
# Phase NN — <title>

**Goal:** <one sentence, copied from phases.md>

## Scope

<What this phase changes. Name the modules and the surfaces.>

## Done criteria

<A numbered list. Each one is observable and testable. Each gets an ID:
DC-1, DC-2. The test plan maps every DC to a test.>

## Not in this phase

## Epics

<One line per epic, in build order, with its file name and one-line goal.>

## Risks
```

## test-plan.md

```markdown
# Test plan — Phase NN <title>

## Coverage

| Done criterion | Proved by | Kind |
|---|---|---|
| DC-1 | <test name or manual step> | automated / manual |

Every done criterion has a row. A criterion with no row is a gap.

## Gaps

<Any done criterion with no test, and the reason. An empty list is the goal.>

## How to run

<The exact command. Copied from the Stack block in architecture.md.>

## Test data

<What the tests need. Fixtures, seeded rows, a test database.>
```

## epic-NN-<slug>.md

The epic file has three blocks. Stage 5 writes the first. Stage 6 appends the
second. Stage 7 appends the third. Nothing rewrites an earlier block.

```markdown
# Epic NN — <title>

**Phase:** NN <title>
**Goal:** <one sentence>
**Done criteria covered:** DC-2, DC-3

## Behaviour

<What the user can do after this epic. Two to four sentences.>

## Acceptance criteria

<Given / When / Then, numbered AC-1, AC-2. Each one is testable.>

- **AC-1** — Given <state>, when <action>, then <result>.

## Tests

<One line per test. The name, what it proves, and which AC it covers.>

## Files

<The files this epic adds or changes. A best guess is fine.>

## Out of scope

## Notes

<Anything the builder needs. Links to ADRs, gotchas in the existing code.>

---

## Build note — <YYYY-MM-DD>

**Result:** green / blocked

### Tasks done

### Tests added

<Each test file and test name.>

### Suite result

<The command run, and the summary line of its output.>

### Deviations from the plan

<What differs from the spec above, and why. "None" is a valid answer.>

### Follow-ups

---

## Review — <YYYY-MM-DD>

**Verdict:** pass / pass with follow-ups / fail

### Acceptance criteria

| AC | Met | Proved by |
|---|---|---|
| AC-1 | yes / no | <test name, or "no test"> |

### Drift

<Anything that contradicts concept.md or architecture.md. "None" is valid.>

### Findings

<Each finding: file and line, what is wrong, and how bad it is.>

### Follow-ups
```

## ADR

`docs/adr/NNNN-<slug>.md`. Offer one only when all three hold: the decision is
hard to reverse, it surprises a reader without the context, and it came from a
real trade-off.

```markdown
# NNNN — <title>

**Status:** accepted
**Date:** <YYYY-MM-DD>

## Context

<What forced the decision.>

## Decision

<What was decided, in the present tense.>

## Consequences

<What this makes easy, and what it makes hard.>

## Alternatives rejected

<Each one, and the reason.>
```

To supersede an ADR, write a new one and set the old status to
`superseded by NNNN`. Never edit the body of an accepted ADR.
