---
name: testradar
description: Find code that has no test and write the missing tests. Use when the user asks "what's untested", "find test gaps", "what needs tests", "write tests for my changes", "am I missing test coverage", or invokes /testradar. Prioritizes recently-changed untested functions and untested branches, detects the project's own test framework and style, shows gaps before writing, and runs the new tests to confirm they pass.
---

# Test Gap Finder + Writer

Two jobs, in order: (1) find the code most worth testing that currently
isn't, and (2) write good tests for the gaps the user approves. Never dump
a wall of low-value tests; the point is meaningful coverage of risky code,
not a coverage-percentage vanity number.

Framework detection and per-language conventions are in
`references/frameworks.md` — read it before writing any test.

## Step 1 — Decide the scope

- Default scope is **what changed**, not the whole repo — untested code you
  just touched is the real risk. Determine changed files from git:
  `git diff --name-only` (unstaged), `git diff --cached --name-only`
  (staged), and `git diff --name-only HEAD~1` if the working tree is clean
  (last commit). Prefer uncommitted changes if any exist.
- If the user explicitly asks for a whole file/module/directory, use that
  scope instead.
- Only consider source files, not test files, config, generated code, or
  vendored dependencies.

## Step 2 — Detect the test setup (don't guess)

From `references/frameworks.md`: identify the language, the test framework
actually in use (look at existing test files and the manifest —
package.json, pom.xml, requirements/pyproject, go.mod, Gemfile, etc.), the
test file naming/location convention, and the assertion + mocking style
THIS project uses. Read 1-2 existing test files to match their idioms
(naming, setup/teardown, fixtures, how they mock). Matching the project's
existing style matters more than any "best practice" — a test that looks
foreign to the codebase is a bad test even if it passes.

If the project has NO tests at all, say so, propose the standard framework
for the language (from the reference), and confirm with the user before
introducing a test dependency — don't silently add a framework.

## Step 3 — Find the gaps and rank them

For each in-scope source unit (function/method/class), determine whether a
test exercises it, and how well:

- **Untested entirely** — no test references it. Highest priority if it has
  real logic (skip trivial getters/setters, pass-throughs, pure config).
- **Partially tested** — a happy-path test exists, but branches are
  uncovered. The uncovered branches are usually the valuable ones:
  - error/exception paths, `catch` blocks
  - boundary conditions (empty, null, zero, max, negative)
  - each side of significant `if`/`switch` branches
  - early returns / guard clauses
- Deprioritize: trivial accessors, generated code, thin delegation, code
  with no branching and no failure mode.

If the project has a coverage tool configured, you may run it for signal
(`references/frameworks.md` lists commands) — but coverage % is an input,
not the goal. A covered line with no assertion about its behavior is still
a gap.

## Step 4 — Present the gaps FIRST (before writing)

Show the user a ranked list: file, unit, what's untested (whole thing / a
specific branch), and why it matters ("no test for the null-input path,
which is the one that throws"). Group by file. Keep it scannable.

Then ask what to write tests for — all of it, the top N, or a selection.
Do NOT write tests before the user has seen the gaps and chosen; a pile of
unsolicited test files is noise, and the gap list itself is valuable even
if they write the tests themselves.

## Step 5 — Write the tests (for approved gaps only)

- One coherent test per behavior/branch, named the way this project names
  tests (mirror existing tests' naming scheme exactly).
- Cover the behavior that matters: the happy path if missing, then each
  meaningful branch — especially the error and edge paths that were the
  reason it was flagged.
- Use the project's real assertion library and mocking approach. Reuse
  existing fixtures/helpers/factories rather than inventing new ones.
- Tests must assert on **behavior**, not implementation details — assert
  what the function returns/throws/does, not how it does it internally, so
  the test survives refactors.
- Never write a test that just re-states the code or asserts trivially
  true things to inflate the count. A test that can't fail is worse than
  no test.
- Put tests in the project's conventional location/file, extending an
  existing test file for that unit if one exists rather than creating a
  duplicate.

## Step 6 — Run them and confirm

Run the newly written tests with the project's test command (from the
reference). They must pass. If one fails:

- If the test is wrong (bad assumption about behavior), fix the test.
- If the test reveals an actual BUG in the code (the behavior is wrong),
  do NOT silently "fix" the test to hide it — stop and tell the user: "this
  test fails because the code appears to do X when Y is expected — is this
  a bug?" A test that surfaces a real bug is the best possible outcome;
  never paper over it.
- Never weaken a test just to make it green.

Report: which gaps were closed, the tests added (file + count), the run
result, and any remaining gaps the user chose not to fill. If a test
uncovered a probable bug, lead with that — it's the most important thing
you found.

## Anti-patterns

- Writing tests before showing the gaps and getting a choice.
- Chasing 100% coverage — testing trivial getters and generated code to
  hit a number. Risk-first, not percentage-first.
- Tests that assert implementation details and break on every refactor.
- Tests that can't fail (tautologies, no real assertion) padding the count.
- Introducing a new test framework/dependency without asking.
- Editing the code under test to make a test pass, when the test actually
  found a bug — surface the bug instead.
