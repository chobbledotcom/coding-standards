---
name: 100% test coverage
subtitle: Every line and every branch, deterministic, with no new exclusions
guide-category: testing
order: 1
meta_description: >-
  How the tickets repository enforces 100% line and branch coverage: an
  exclusion list that only shrinks, in-process tests for subprocess
  branches, code shaped for the coverage merge, and per-file coverage
  while you work.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      [Test coverage](/glossary/#coverage) measures which lines and *branches* (each
      way through an `if`) ran during the tests. Read
      [Deno's coverage docs](https://docs.deno.com/runtime/reference/cli/coverage/)
      or [Bun's](https://bun.sh/docs/test/coverage) to see how to produce a
      report, and [Martin Fowler on test coverage](https://martinfowler.com/bliki/TestCoverage.html)
      for why a high number on its own is not enough.
  - type: markdown
    content: |
      ## Why 100% and not 90%

      At 90%, the question for every uncovered line is "is this one of the
      10% we accept?", and an agent will always answer yes. At 100%, there
      is no question. Every uncovered line is a finding, and the only
      answers are a test or a deletion.

      Full coverage is a floor, not a goal. It does not prove the tests
      check anything; that is what
      [mutation testing](/techniques/testing/mutation-testing/) and the
      [assertion gate](/techniques/testing/test-quality/) are for. What it
      does prove is that no line of code ships without having run at least
      once under test, and it has a useful side effect: code that is hard
      to cover is usually code that should not exist.

      ## The rules

      From AGENTS.md:

      > **100% test coverage**: Coverage must be complete and deterministic.
      > A branch that only a spawned subprocess reaches gets a direct
      > in-process unit test.
      >
      > **Never add a coverage exclusion**: The list in
      > [`scripts/check-coverage-exclusions/exclusions.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-coverage-exclusions/exclusions.ts) only ever shrinks,
      > and the ratchet fails on an entry a branch adds. Answer a coverage
      > gap by restructuring the code.

      The gate checks lines *and* branches, per file, from the [lcov](https://github.com/linux-test-project/lcov) report
      ([`scripts/coverage-check.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/coverage-check.ts)).
      The [chobble-template](https://github.com/chobbledotcom/chobble-template)
      does the same through Bun, with
      `coverageThreshold = { lines = 1.0, functions = 1.0 }` in
      [`bunfig.toml`](https://github.com/chobbledotcom/chobble-template/blob/main/bunfig.toml).

      ## Deterministic means no lucky runs

      A line that is covered on some runs and not others is a flaky gate.
      Two common causes, and the rules that remove them:

      - **Subprocess coverage.** Code reached only by a spawned child
        process may or may not be counted, depending on the tooling. So the
        rule is that such a branch also gets a direct, in-process unit test.
      - **Timing.** A retry path that only runs when something is slow is
        covered by luck. Retry paths run under a virtual clock instead
        ([`withVirtualBackoff`](https://github.com/chobbledotcom/tickets/blob/main/test/test-utils/virtual-time.ts#L11) from the test utilities), so the "every retry
        failed" branch runs on every run, instantly.

      ## Code shaped for the coverage merge

      The test runner runs files in parallel isolates and merges their
      coverage. Three times, that merge reported a function's lines as
      unhit while the tests clearly ran them. One case had a known shape:
      the logic sat in the inner arrow of a curried ternary helper.
      Rewritten as a plain two-argument function with an `if`/`return` body,
      it passed with no exclusion
      ([#2504](https://github.com/chobbledotcom/tickets/pull/2504)).

      So the design guide asks for a
      ["house shape"](https://github.com/chobbledotcom/tickets/blob/main/docs/designing-systems.md#readable-by-the-coverage-merge):
      logic in a flat, named, module-level function, and currying from a
      shared combinator rather than hand-written closures.

      ```typescript
      // src/shared/types.ts: the logic is a plain two-argument function.
      const hasRole = (levels: readonly AdminLevel[], level: AdminLevel): boolean =>
        levels.includes(level);

      // Currying comes from the shared combinator in #shared/fp-bind.ts.
      const roleIn = bindFirst(hasRole);

      export const isStaffRole = roleIn(STAFF_ADMIN_LEVELS);
      export const isOwnerRole = roleIn(OWNER_ADMIN_LEVELS);
      ```

      The document is careful to say this is a working rule from one case,
      not a proven cause. That honesty matters: the next agent to hit the
      problem knows what is known and what is guessed.

      ## Coverage while you work

      Running the full suite with coverage on a shared machine is slow and
      unkind to the other agents. [`deno task cov:files`](https://github.com/chobbledotcom/tickets/blob/main/scripts/cov-files.ts) runs only the given
      test files with the full harness, then prints the lines and branches
      that run left uncovered, optionally limited to one source file with
      `--only`. An agent can close a gap in a minute and leave the full
      gate to CI.

  - type: callout
    variant: warning
    icon: "mdi:alert-outline"
    name: The exclusion list is the weak point
    content: |
      Every coverage tool has an ignore list, and every agent finds it. In
      the template the exclusions are in `bunfig.toml` with a reason per
      entry. In tickets they are in a [TypeScript](https://www.typescriptlang.org/docs/) file with a check that
      diffs it against `origin/main` and fails on any addition. The second
      is the one that holds.
---
