---
name: Mutation testing
subtitle: Change the code on purpose, and check that a test notices
guide-category: testing
order: 3
meta_description: >-
  How mutation testing works in the tickets and chobble-template
  repositories: an in-house runner, lint and type-check gates before tests,
  isolated copies, a 100% kill rate on changed files, and equivalent
  mutants with re-audit stamps.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Tests for your tests

      Coverage tells you a line ran. It does not tell you that any test
      would fail if the line were wrong. Mutation testing does. The runner
      makes one small change to the source (a *mutant*), for example `<`
      becomes `<=`, `&&` becomes `||`, `+` becomes `-`, and runs the tests.
      If a test fails, the mutant is *killed*. If every test still passes,
      the mutant *survived*, and you have found a change to your code that
      nothing would notice.

      A survivor reads like this:

      ```text
      src/shared/dates.ts:42:17  >= → >
      ```

      That is a precise, actionable finding: there is no test for the
      boundary on line 42. Agents are good at fixing findings this precise.

      ## The bar: 100% on the code you changed

      In the tickets repository the policy is:

      > Survivors on a file you touch are yours to fix, even on lines you did
      > not change. Write the assertion that kills the mutant, or record it
      > in `scripts/mutation/equivalent-mutants/` with a proof that no input
      > can distinguish it.

      `deno task precommit:mutation` is the branch gate. It mutates every
      `src/` file the branch changed, against `origin/main`, and demands a
      100% kill rate. It runs the mirror-located direct tests first, then
      the changed integration tests and Cucumber specs only for whatever
      survived. Because it only covers changed files, its cost grows with
      the size of the change, not the size of the codebase.

      ```bash
      deno task mutation src/shared/dates.ts test/shared/dates.test.ts
      deno task mutation 'src/shared/forms/definition.ts' \
        'test/shared/forms/definition/*.test.ts' --exhaustive
      deno task precommit:mutation
      ```

      ## Cheap gates first

      Running the test suite once per mutant is expensive. The runner puts
      every mutant through two cheaper gates first, from
      [docs/mutation-testing.md](https://github.com/chobbledotcom/tickets/blob/main/docs/mutation-testing.md):

      1. **Lint.** A per-file Biome run. A mutant that turns `===` into `==`
         breaks `noDoubleEquals`, so it could never ship. Killed.
      2. **Type-check.** `deno check` on the mutant. A mutant that turns
         `"a" + "b"` into `"a" * "b"` is a type error. Killed.

      Only mutants that pass both reach the tests. Each gate is trusted only
      after the runner confirms the *unmutated* file passes it, so a file
      that was already broken aborts the run instead of scoring a fake 100%.

      ## No mutant is judged by a clock

      An early version timed out slow mutants and counted them as killed.
      That made a slow type-check look like a caught bug. Now gates and
      tests always run to completion, and the only clock is a whole-run
      deadline (one hour by default) that fails the run with no score.

      > No mutant is ever judged by a clock: gates and tests run to
      > completion, so a mutant is killed only when a gate rejects it or a
      > test fails, and survives otherwise.

      ## Isolated copies

      The runner copies the checkout to `.mutation-runs/<id>/work` and
      mutates files *in place inside the copy*. Two reasons:

      - In-place mutation is what makes the mutant reach code imported
        through `#` import-map aliases. The original tool the operator
        tables came from,
        [Mutasaurus](https://github.com/christoshrousis/mutasaurus), wrote a
        temporary copy but ran the original tests, so on an alias-based
        project every mutant falsely survived.
      - Several agents share the machine. A mutant must never be visible in
        the live checkout, where another process could commit it.

      `deno task mutation --list`, `--kill` and `--clean` manage runs, and
      every run cleans up after earlier runs that are no longer going.

      ## Equivalent mutants, with an expiry date

      Some mutants cannot be killed because they do not change behaviour.
      If every caller passes a non-empty string, `?? "x"` and `|| "x"` are
      the same. These go in a registry with a written proof:

      ```text
      src/shared/db/attendees/select.ts::whereClauses.parts.clause~0zb4tlc  ?? → || audited:0g48txk
        # where.kind: AttendeeKindFilter|undefined — every member is truthy,
        # so `?? "attendee"` and `|| "attendee"` agree on every input
      ```

      The `audited:` token is a hash of the source file at the moment a
      person last re-derived the proof. When the file changes, the entry
      becomes **unconfirmed**: it suppresses nothing, the mutant runs again,
      and the run fails until someone re-checks the proof or writes the
      test. A proof about old code cannot hide a real survivor in new code.

      `deno task mutation:audit-equivalents --tests` goes further: it tries
      to kill every recorded entry with the current tests, and drops any
      entry a test kills, because a killed mutant was never equivalent.

      ## Time-box it

      > A manual run must finish in about ten minutes. When a run needs
      > longer, stop it, and rely on the direct tests around the method you
      > changed. A small edit owes those tests, not a mutation run.

      Mutation testing is the most expensive check in the repository, so it
      is the one most in need of a stopping rule.

  - type: callout
    variant: warning
    icon: "mdi:alert-outline"
    name: What mutation testing cannot do
    content: |
      "Do not use mutation testing to invent the behavior contract. It can
      show that an existing assertion is weak; it cannot show that a
      missing state, race, or external failure was never designed." The
      [behaviour contract](/techniques/instructing-agents/behaviour-contracts/)
      comes first.

  - type: markdown
    content: |
      ## In the chobble-template

      The template has its own runner (`bun run mutation`), built from the
      same Mutasaurus operator tables, and a plain-text list of
      confirmed-equivalent survivors in
      `scripts/mutation/equivalent-mutants.txt` so that CI can gate on new
      gaps only. Both repositories also have a manual GitHub workflow that
      mutation-tests a chosen module on demand.
---
