---
name: design
description: Stage 3b of the SDLC chain, optional. Settles the user interface before any code exists — surfaces, tokens, components, and states. Writes docs/sdlc/design.md, then offers to generate screens with the Stitch MCP and, last, to push a component kit to Claude Design. Use when the user says "design the UI", "what should this look like", "make the screens", "design system", "mockups", or has an approved architecture for a project with a user interface. Do NOT use for a project with no UI, and do NOT use to write frontend code — that is stage 6, /sdlc:build-epic.
argument-hint: "[optional: a look, a reference URL, or a screenshot path]"
---

# Design: settle the interface before the code

**Reference given:** $ARGUMENTS

Optional stage. The chain runs without it. It earns its place only when the
project has a user interface worth arguing about.

## Preconditions

Read `docs/sdlc/state.md`. The `architecture` stage must be `approved`. If it
is not, name the missing stage and stop.

Read `docs/sdlc/architecture.md` in full. The `## Stack` block decides
everything here. A Jinja2 and HTMX project takes server-rendered templates. A
React project takes components. Design for the stack that exists.

Read `docs/sdlc/concept.md`. The screens come from what the user must do.

## First, does this stage apply?

End your first message with this line, and nothing after it:

> **Does this project have a user interface that a person looks at?** Reply
> `yes`, or `no` to skip this stage.

If the answer is no, say this stage does not apply, set nothing, and close with
`/sdlc:phases`. Do not write `design.md` for a library or a CLI.

A CLI with rich terminal output is a borderline case. Run the stage only when
the user asks for it.

## Orient on what exists

For an existing repo, read before you propose:

- The template or component directory.
- The stylesheet, the Tailwind config, or the token file.
- Any screenshot or export the user passed in `$ARGUMENTS`.

Say what you keep and what you replace. A brownfield design starts from the
current look, not from a blank page.

## What to settle

Cover these, one question at a time. Skip any the concept already fixed.

1. **Surfaces.** Which screens exist? Name each one and what it is for. Six or
   fewer for a first phase.
2. **The primary flow.** The one path that proves the concept. Walk it screen
   by screen.
3. **Look.** Two or three words the user agrees to. "Dense and technical", not
   "clean and modern". Every project is clean and modern.
4. **Tokens.** Colour, type scale, spacing, radius. Name the values.
5. **Components.** The parts that repeat. Name each one and its variants.
6. **States.** Empty, loading, error, and no-permission. A screen with no empty
   state is unfinished.
7. **Responsive.** Which widths matter. Say the smallest one you support.

Do not design what the phase list does not build. This stage covers the same
scope as the concept.

## Writing the document

Write `docs/sdlc/design.md` with the template in
[ARTIFACTS.md](../status/ARTIFACTS.md).

That file has two jobs. Stage 6 reads it to build the UI, and Stitch reads it
as its `DESIGN.md`. Keep the token block literal, with real values, so both
readers get the same answer.

Then update `docs/sdlc/state.md`:

- Set `design` to `in progress`.

## Generating screens with Stitch

Offer this. Never run it without an answer, because it calls a network service
and creates a project on it.

End the message with:

> WARNING: This creates a project on Stitch, an external service, and sends it
> the contents of `design.md`.
>
> **Generate these screens in Stitch?** Reply `yes`, or `skip` to move on.

On yes:

1. `mcp__stitch__list_projects`. Reuse a project for this repo when one exists.
   Otherwise `mcp__stitch__create_project`.
2. `mcp__stitch__upload_design_md` with `design.md`, then
   `mcp__stitch__create_design_system_from_design_md`. The design system comes
   from the tokens you just agreed, so every screen matches.
3. `mcp__stitch__generate_screen_from_text` for each surface, one call per
   screen, always with the `designSystem` id from step 2. Generation is slow.
   Do not retry a timeout. Poll `mcp__stitch__get_screen` instead.
4. Record the project id and the design system id in `design.md`, under
   `## Where the screens live`.

Show the user the screens, then end the message with: **What do you want
changed?** Name a screen and the change, or reply `none`.

Use `mcp__stitch__edit_screens` for a fix and `mcp__stitch__generate_variants`
when the user wants options.

If the Stitch MCP is not connected, say so in one line, and continue. The
document is the deliverable. The screens are a convenience.

## Pushing a component kit to Claude Design

Offer this last, and only when the user has agreed to the tokens and the
component list.

Claude Design holds a rendered view of a component library at
`claude.ai/design`. It is a shared reference. It is not the source of truth —
the repo is.

End the message with:

> WARNING: This uploads the component kit to your `claude.ai/design` account.
>
> **Push a component kit to Claude Design?** Reply `yes`, or `skip` to finish
> without it.

On yes:

1. `DesignSync` with `list_projects`. Pick an existing design-system project or
   `create_project`.
2. Build the kit on disk first, under `docs/sdlc/design-kit/`. One preview HTML
   file per component, plus a tokens file.
3. `finalize_plan` with the exact paths, then `write_files` with `localPath`
   for each file.
4. Record the project id in `design.md`.

The tool needs a claude.ai login with design scopes. It prompts on the first
call. If the tool is absent or the login fails, say so in one line and stop
that step. Nothing else in the stage depends on it.

Never do a wholesale replace of a Claude Design project. Sync the components
you changed.

## The gate

Summarise in five bullets or fewer: the surfaces, the look in three words, the
token decisions, and where the screens live. Give the path. Do not paste the
document into the chat.

Then end the message with this line, and nothing after it:

> **Approve `design`?** Reply `yes`, or tell me what to change.

No `Next` block follows the question. See
[ASKING.md](../status/ASKING.md).

On approval, set `design` to `approved <today's date>`.

## Close

End with the block from [NEXT-STEPS.md](../status/NEXT-STEPS.md). After
approval it reads:

```
Next
  → /sdlc:phases           decide the build order
    /sdlc:architect        the stack fights the design
```

## What this skill does not do

- It does not write frontend code, templates, or CSS in the app. Stage 6 does
  that, from this document.
- It does not install a dependency.
- It does not replace a Claude Design project wholesale.
- It does not run without an approved architecture.
- It does not commit.

## Reference

- [ASKING.md](../status/ASKING.md) — how to ask. This stage asks four times,
  in four messages. Never bundle them.
- [ARTIFACTS.md](../status/ARTIFACTS.md) — the `design.md` template.
- [STATE-FORMAT.md](../status/STATE-FORMAT.md) — the state file and its rules.
