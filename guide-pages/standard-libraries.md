---
name: Libraries over hand-rolled code
subtitle: Move home-made validators and helpers onto Valibot and the Deno standard library
guide-category: instructing-agents
order: 6
meta_description: >-
  How the tickets repository replaced hand-rolled regular expressions,
  sum loops and byte converters with Valibot, @std/collections and
  runtime built-ins, and why that matters more when agents write the code.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A **standard library** is the set of well-tested modules that ships
      with, or is maintained alongside, a language or runtime. Deno's lives
      at [jsr.io/@std](https://jsr.io/@std), and each module is imported
      by name, for example `@std/collections`. [Valibot](/glossary/#schema)
      is a small library for [schema validation](/glossary/#schema). Read
      the [Valibot introduction](https://valibot.dev/guides/introduction/)
      and skim the
      [`@std/collections` function list](https://jsr.io/@std/collections/doc)
      before you read this page.
  - type: markdown
    content: |
      ## Agents love to write it themselves

      Ask an agent to check an email address and it writes a regular
      expression. Ask it to add up some prices and it writes a `reduce`.
      Ask it to turn bytes into base64 and it writes a loop over
      `String.fromCharCode`. Each one works on the inputs the agent tried.

      The trouble shows up later. Every hand-rolled helper is code you own:
      it needs tests, it needs [mutation-hardening](/techniques/testing/mutation-testing/),
      and it drifts. Four slightly different date checks in four files are
      four places to get a bug, and four near-copies for
      [jscpd](/techniques/static-checks/jscpd/) to find.

      A library function has been used by thousands of other projects. Its
      edge cases are already found. You test that you call it correctly,
      not that it works.

      ## The rule in AGENTS.md

      The tickets
      [AGENTS.md](https://github.com/chobbledotcom/tickets/blob/main/AGENTS.md)
      names the libraries outright, so an agent does not have to guess:

      > Validate external data at the boundary with a valibot schema, or
      > check and throw the way `parseMessageId` does.

      > For an operation `#fp` does not cover, use `@std/collections`
      > directly, and wrap it in a curried `#fp` adapter once more than one
      > caller needs it.

      [docs/designing-systems.md](https://github.com/chobbledotcom/tickets/blob/main/docs/designing-systems.md)
      lists "built on valibot and the standard libraries" as one of the
      qualities every new system must have (see
      [Exemplars over essays](/techniques/instructing-agents/exemplars/)).

  - type: split-code
    subtitle: Validators
    content: |
      ## From regular expressions to Valibot

      The first move was email
      ([#1163](https://github.com/chobbledotcom/tickets/pull/1163)). A
      single `EMAIL_REGEX` became a Valibot schema that trims, lowercases,
      checks and *brands* the value. A function that takes a `ValidEmail`
      can only receive an address that passed the check.

      The next day the rest followed, one small pull request each:

      - **Slugs** moved to Valibot's `v.slug()`
        ([#1171](https://github.com/chobbledotcom/tickets/pull/1171)).
      - **Dates** in `YYYY-MM-DD` form had a home-made regex in four
        places, each a little different. All four became one `isIsoDate`
        helper on Valibot's `isoDate`
        ([#1172](https://github.com/chobbledotcom/tickets/pull/1172)). The
        old `\d{2}` pattern accepted `2026-99-99`. The new one does not.
      - **Case conversion** for database column names moved to
        `toCamelCase` and `toSnakeCase`. The email provider guard became a
        `picklist`
        ([#1173](https://github.com/chobbledotcom/tickets/pull/1173)).
      - **Tests** check API responses with a `strictObject` schema
        ([#1178](https://github.com/chobbledotcom/tickets/pull/1178)). One
        parse replaced seventeen "does not expose this field" assertions.

      Today more than a hundred files under `src/` import Valibot.
    figure_filename: src/shared/validation/email.ts
    figure_language: typescript
    figure_code: |
      // Before (#1163):
      export const EMAIL_REGEX =
        /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

      // After:
      export const EmailSchema = v.pipe(
        v.string(),
        v.trim(),
        v.toLowerCase(),
        v.email(),
        v.brand("ValidEmail"),
      );

  - type: split-code
    subtitle: Collections
    content: |
      ## From loops to `@std/collections`

      The codebase is functional and curried, built on a small
      [`#fp`](https://github.com/chobbledotcom/tickets/blob/main/src/fp.ts)
      module. That module used to hand-roll everything.
      [#1205](https://github.com/chobbledotcom/tickets/pull/1205) rewired
      it onto
      [`@std/collections`](https://jsr.io/@std/collections/doc):

      - `unique`, `uniqueBy` and `chunk` now call `distinct`, `distinctBy`
        and `chunk`. The std `chunk` throws on a size below one. The old
        version looped for ever.
      - A new curried `sumOf` replaced about twenty `reduce` sums.
      - A new `mapNotNullish` replaced `compact(map(...))` chains.

      The house style stays the same. `#fp` keeps its curried,
      pipe-friendly shape, and each function is a thin adapter. Where std
      has no match, the code says so: the numeric `range` helper is still
      hand-rolled, and its doc comment explains that `@std/collections` has
      no numeric range.
    figure_filename: src/fp.ts
    figure_language: typescript
    figure_code: |
      /**
       * Curried sum-by-selector. Curried adapter
       * over `@std/collections.sumOf`.
       * Replaces the common pattern:
       * reduce((acc, x) => acc + selector(x), 0)
       */
      export const sumOf =
        <T>(selector: (item: T) => number) =>
        (array: Iterable<T>): number =>
          stdSumOf(array, selector);

  - type: markdown
    content: |
      ## From helpers to the runtime itself

      Sometimes the best library is the one already in the runtime.
      [#1175](https://github.com/chobbledotcom/tickets/pull/1175) removed
      hand-rolled byte loops in five files in favour of the native
      `Uint8Array` methods `toBase64()`, `toHex()` and `fromBase64()`. The
      output stayed byte-identical over 5,000 random inputs, and the old
      `String.fromCharCode(...bytes)` calls, which could overflow the stack
      on a large buffer, went away. The same change swapped
      `[...array].sort()` for `toSorted()` and a push loop for
      `Array.fromAsync()`.

      Dates went the same way.
      [#1321](https://github.com/chobbledotcom/tickets/pull/1321) replaced
      a date library with the built-in
      [`Temporal`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal)
      API, which removed a runtime dependency with no change in behaviour.

      The test suite moved too. In October a long series of pull requests,
      for example [#2686](https://github.com/chobbledotcom/tickets/pull/2686)
      and [#2625](https://github.com/chobbledotcom/tickets/pull/2625),
      retired a custom `toContain` matcher. More than 5,000 call sites in
      the UI tests alone now use `assertStringIncludes` and
      `assertArrayIncludes` from
      [`@std/assert`](https://jsr.io/@std/assert/doc).

      ## How to do a migration like this safely

      Each of these pull requests followed the same steps:

      1. **Pick one kind of helper.** Email, then slugs, then dates. Do not
         mix them in one change.
      2. **Prove the outputs match.** The case-conversion change compared
         every real column name in both directions. The base64 change
         compared 5,000 random inputs.
      3. **Say out loud where the library is stricter.** Valibot's email
         check rejects `admin!@example.com`, which the old regex allowed.
         The commit message says so, and the tests changed to match.
      4. **Delete the old helper in the same change.** No wrapper that
         keeps the old name alive. AGENTS.md calls this "no internal
         compatibility layers".
      5. **Move the edge-case tests to one place.** After #1172, one
         `isIsoDate` test covers every date format case, and the call
         sites only test that they call it.

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      Search your codebase for regular expressions, `reduce` calls that
      add numbers, and loops over bytes or characters. For each one, ask
      whether your runtime or a well-known library already has it. Then
      write the library's name into your AGENTS.md, so the next agent
      reaches for it first.
---
