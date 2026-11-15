# The loop

Red, green, refactor, one vertical slice at a time.

## Vertical, never horizontal

Wrong — every layer built separately, nothing works until the end:

```
  models      ████████████████
  services              ████████████████
  routes                          ████████████████
  templates                                 ████████████
  ↑ nothing runs until here
```

Right — a thin line through every layer, then the next line:

```
  slice 1   ██   works
  slice 2     ██   works
  slice 3       ██   works
```

Slice 1 is the tracer bullet. One route, one handler, one query, one template,
one test. It proves the layers connect. Every later slice widens it.

A slice that touches only one layer is not a slice. Merge it into the slice
that uses it.

## The cycle

Per slice:

1. **Red.** In statically typed or cross-layer codebases, verify exact existing
   enum/record/module signatures first — or compile minimal unimplemented
   public type/interface stubs as part of Slice 1 RED — before generating
   multi-test integration suites against new types/DTOs, so you do not guess
   names and burn compile-fix retries. Write one failing test. Run it. Read the
   failure message. Make sure it fails for the reason you expect. A test that
   fails on an import error or guessed symbol proves nothing.
2. **Green.** Write the smallest code that passes. Hard-coded values are fine
   in slice 1. The next test removes them.
3. **Verify.** Use filtered incremental test runs (`--filter` / `--no-restore`
   after edits; pass `--no-build` only when binaries were already built for the
   current edits) for inner-loop slices, and run the whole suite plus any
   batched headless/runtime verification scripts once at final GREEN. A slice
   is not green if it reddens another test.
4. **Refactor.** Improve the names and the shape while tests stay green. Re-run
   relevant tests after each change, and verify the full suite is green before
   finishing. Stop when the code reads well.

Never write behaviour before the test (minimal unimplemented public
type/interface stubs in Slice 1 RED are fine so tests compile). Keep RED
focused on one slice at a time rather than dumping speculative tests across
unbuilt layers.

## Where to put the test

Use the highest seam that still proves the behaviour.

- A test through the HTTP route beats four tests on the functions behind it.
- A test through the public function beats a test on a private helper.
- Test a private helper only when its logic is hard and the public path cannot
  reach every case.

The acceptance criterion tells you the seam. `Given a logged-out user, when
they open /admin, then they see a 403` is a route test.

Name the test after the behaviour, not the function. `test_malformed_row_is_
reported`, not `test_parse_row_2`.

Use the project's domain vocabulary. If the `CONTEXT.md` glossary calls it a
`source`, the test says `source`.

## Mocking

Mock at the edge of your system, never inside it.

- **Mock:** a third-party HTTP API, a clock, a random seed, an email sender, a
  payment provider.
- **Do not mock:** your own functions, your own classes, your own database.

A test that mocks your own code tests the mock. When mocking your own code
feels necessary, the seam is in the wrong place. Move the seam.

**When the project uses a database, it is not a third-party service.** Use a
dedicated real test database (for example `TEST_DATABASE_URL` from the project
configuration), separate from any live database. Never point a test at a live
database.

## Definition of green

Green means all four:

1. Every test in the suite passes, including the tests that existed before.
2. No test is skipped, marked expected-to-fail, or commented out.
3. The type check and the linter pass, when the project has them.
4. Every acceptance criterion in the epic has at least one passing test.

Three failures of the same test in a row is the stop condition. Report what
failed, what you tried, and what you think is wrong. Do not delete the test.

## Refactor rules

Refactor only on green. Refactor in small steps. Re-run relevant tests after
each one, and confirm the full suite stays green.

Worth doing:

- A name that does not say what the thing is.
- Duplication that appeared three times.
- A function that needs a comment to explain its middle.

Not worth doing now:

- A pattern for a need that has not appeared yet.
- A rewrite of code this epic does not touch.
- A dependency swap.

Put those in `### Follow-ups` in the build note instead.

## Deviating from the plan

The plan is a good guess, not a contract. Code teaches you things the plan
could not know.

When a slice cannot work as planned:

1. Do the smallest thing that satisfies the acceptance criterion.
2. Write the deviation and its reason in the build note.
3. Keep going.

When the deviation contradicts `architecture.md`, stop instead. That is a
decision for the user, not for the loop.
