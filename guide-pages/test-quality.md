---
name: Test quality gates
subtitle: Block the assertions that pass for almost any bug
guide-category: testing
order: 2
meta_description: >-
  How weak tests are caught: a weak-assertion gate on every touched test
  file, a registry that blocks toBeDefined, test quality criteria, and the
  rules that keep the suite fast and deterministic.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      An [assertion](/glossary/#assertion) is the line that checks a test's result.
      The examples use the `expect` style from
      [Deno's standard library](https://jsr.io/@std/expect), which matches
      [Jest](https://jestjs.io/docs/expect). A *tautological* test checks a
      value it set itself, so it can never fail. A *regression test* reproduces
      a bug that was fixed, so the bug cannot come back unnoticed.
  - type: markdown
    content: |
      ## The soft test

      An agent writing tests to reach coverage writes the cheapest assertion
      that passes. Very often that is:

      ```typescript
      const result = await createBooking(input);
      expect(result).toBeDefined();
      expect(result.ok).toBeTruthy();
      ```

      This runs every line of `createBooking` and checks almost nothing.
      Return the wrong attendee, the wrong amount or the wrong state, and it
      still passes.

      ## What a strong test asserts

      AGENTS.md puts it simply:

      > Prefer assertions that fail under realistic mutants: exact values,
      > object shape, persisted rows. Avoid `toBeTruthy()`, compound
      > booleans, and presence-only checks.

      And for critical flows it asks for the cases agents skip: negative
      paths, idempotency, concurrency and metamorphic cases. "Webhook replay
      does not double-credit, capacity cannot go below zero, role
      downgrades remove access, PII stays encrypted or absent from
      responses."

      ## The weak-assertion gate

      [`deno task check:test-quality`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-test-quality.ts) scans every test file that the branch
      touches, compared with the merge base on `origin/main`, for patterns
      like these
      ([`scripts/test-quality-audit-model.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/test-quality-audit-model.ts)):

      ```typescript
      // presence-only
      /expect\s*\([^;]+?\)\s*\.\s*toBe(?:Defined|Undefined)\s*\(/g
      // truthiness
      /expect\s*\([^;]+?\)\s*\.\s*toBe(?:Truthy|Falsy)\s*\(/g
      // a comparison squeezed into a boolean
      /expect\s*\([^)]*(?:&&|\|\||===|!==|>=|<=|>|<)[^)]*\)\s*\.\s*toBe\s*\(\s*(?:true|false)\s*\)/g
      ```

      The scope is the clever part. A file the branch does not touch is
      left alone. A file the branch *does* touch must hold no weak assertion
      at all, whether the branch added it or inherited it:

      > A test file the branch touches must hold no weak assertion at all
      > [...] Removing a weak assertion is the only way out.

      That turns every change into a small clean-up of the files it
      touches, without forcing a big-bang rewrite of the whole suite.

      ## `toBeDefined()` is blocked outright

      Presence assertions get their own ratchet. `deno task
      check:presence-assertions` holds every test file to its recorded count
      of `toBeDefined()` in a registry, and the count only falls. A new one
      anywhere fails the build. `toBeUndefined()` stays allowed where the
      code documents an absent value.

      ## The template's test quality criteria

      The chobble-template has a checklist every test must meet,
      [TEST-QUALITY-CRITERIA.md](https://github.com/chobbledotcom/chobble-template/blob/main/test/TEST-QUALITY-CRITERIA.md).
      The first three items are the ones agents fail most:

      1. **Tests production code, not reimplementations.** No logic copied
         from production into the test. Constants are imported, not
         re-typed.
      2. **Not tautological.** The test does not assert a value it just
         set. Production code runs between setup and assertion.
      3. **Tests behaviour, not implementation details.** It checks
         observable outcomes, not internal state.

      Each item has a bad and a good example. The bad one for "not
      tautological":

      ```javascript
      // This tests nothing - you set it, then check you set it
      button.style.display = "none";
      assert.strictEqual(button.style.display, "none");
      ```

      ## Regression tests come first

      > **Every bug fix ships with a regression test**: Write the failing
      > test first, confirm it fails for the right reason, then fix and
      > watch it go green. The test must reproduce the real bug.

      "For the right reason" matters. An agent will happily write a test
      that fails because of a typo, fix the typo along with the bug, and
      call it a regression test.

      ## Fast and honest

      A slow suite gets skipped, and a flaky one gets retried until it
      lies. The suite rules:

      - Every test slower than 500 ms is printed after each run, and
        treated as a regression to fix.
      - Never sleep for real. Retry paths run under a virtual clock.
      - Never run repo tooling as a subprocess inside a test.
      - Render a page once and assert many things about it.
      - Seed volume with one batch insert, not a loop.
      - Test files share isolates, so a test resets the state it changes
        and never relies on a fresh process. A file that truly needs its
        own isolate says so with a `// test-groups: run-alone` marker.
      - A test that reserves a real port or file must tell a stolen
        resource from its own, because other suites run in parallel on the
        same machine.
---
