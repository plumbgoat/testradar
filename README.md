# testradar

Find the code you just changed that has **no test** — then write the
missing tests, in your project's own framework and style.

```
/testradar
```

**Who it's for:** any developer who ships changes and isn't sure what
they left untested — especially before opening a PR.

## What it does

1. **Looks at what you changed** (uncommitted, staged, or your last
   commit) — not the whole repo, because untested code you *just touched*
   is the real risk.
2. **Detects your test setup** — framework, file locations, naming, and
   assertion/mocking style — by reading your existing tests, and matches
   them. It won't impose a foreign style or add a framework without asking.
3. **Finds the gaps and ranks them** — fully untested functions first,
   then the untested *branches* inside functions that look covered (the
   error paths and edge cases that a happy-path test skips — usually the
   ones that actually break).
4. **Shows you the gaps before writing anything**, so you choose what's
   worth testing — all of it, the top few, or a selection.
5. **Writes the tests** for what you approve, matching your codebase's
   idioms, asserting on behavior (not internals, so they survive
   refactors).
6. **Runs them to confirm they pass** — and if a test uncovers a real bug
   instead, it tells you rather than quietly hiding it.

## What makes it different

- **Risk-first, not percentage-first.** It won't write pointless tests for
  trivial getters to inflate a coverage number. It targets code with a
  failure mode.
- **Gaps first, tests second.** The list of what's untested is valuable on
  its own — you're never buried under unsolicited test files.
- **Finds bugs, doesn't hide them.** If a new test fails because the code
  is wrong, that's surfaced as the headline — not papered over to go green.

## Install

```
/plugin marketplace add plumbgoat/testradar
/plugin install testradar@testradar-marketplace
```

## Use

```
/testradar                     # what did I change that isn't tested?
/testradar the whole payments module
```

or just ask: *"what did I change that isn't tested — and write the tests."*

## Safety

It writes test files and runs the test suite. It won't change the code
under test — except that if a test reveals a bug, it stops and asks you,
rather than editing the code or weakening the test to force a pass.

## License

MIT
