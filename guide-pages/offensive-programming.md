---
name: Offensive programming
subtitle: Never suppress an error, never default away a missing value
guide-category: static-checks
order: 5
meta_description: >-
  The fail-loudly rules and the checks behind them: no empty catch blocks,
  no unchecked double casts, no fallback operators hiding missing data,
  OrNull naming for documented absence, and valibot at every boundary.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## The fastest way to a green test

      When an agent's test fails with `TypeError: cannot read properties of
      undefined`, the quickest fix is `?.`. When a call throws, the quickest
      fix is `try { … } catch {}`. When a value might be missing, the
      quickest fix is `?? []`. Each of these makes the test pass and turns a
      loud bug into a quiet wrong answer.

      AGENTS.md has a whole section against this, titled
      **Offensive Programming — Never Suppress Errors**:

      > Fail loudly and immediately. Never suppress, default away, or paper
      > over an error.

      ## The rules

      - **Let errors propagate.** No `try`/`catch` "for extra safety".
      - **Validate external data at the boundary.** "A missing expected
        field from structured external data is a hard no to default away."
        Use a valibot schema, or check and throw.
      - **Fallback operators are for genuinely optional values only.** "Do
        not use `??` / `||` / `?.` to make a missing value someone else's
        problem."
      - **Casts are claims.** "An unchecked `!` or `as` cast claims a shape
        that nothing checked. Parse, do not pretend."
      - **No empty catch.** Catch only where there is a real recovery path.
        A `catch {}` is allowed only when the fallback is the documented
        behaviour, stated in a comment.
      - **Lookups throw.** "A function that looks something up must throw
        when it cannot. Never return `null`, `""`, `0`, or `[]` as a 'not
        found' stand-in." When absence is a real outcome, the function is
        named `…OrNull`, returns `| null`, and a comment says why.

      There is a companion rule about scope, so this does not turn into
      paranoia: **Do not defend against the impossible.** No fallbacks for
      failures that need the whole app to be down. And: **A loud database
      failure is enough.** Do not build rescue paths around a failed write;
      let the full error reach the log.

  - type: split-code
    subtitle: Check and throw
    content: |
      ## What a boundary looks like

      External data is parsed once, where it enters. A missing field is a
      thrown error with a message that names what was missing, not a
      default that travels inward and surfaces three layers later.

      Downstream code then takes the typed value and never re-checks it.
    figure_filename: src/shared/validation/email.ts
    figure_language: typescript
    figure_code: |
      // The output is branded as ValidEmail, so a
      // value can only be produced by passing
      // validation.
      export const EmailSchema = v.pipe(
        v.string(),
        v.trim(),
        v.toLowerCase(),
        v.email(),
        v.brand("ValidEmail"),
      )

  - type: markdown
    content: |
      ## The checks

      | Check | Where | What it finds |
      | --- | --- | --- |
      | `check:empty-catch` | tickets | Every catch block must recover, re-raise, or comment its fallback |
      | `check:double-casts` | tickets | `as unknown as` casts, against a per-file baseline that only falls |
      | `or-fallbacks.test.js` | template | `\|\| []`, `\|\| {}`, `\|\| "…"` outside the data layer |
      | `nullish-coalescing.test.js` | template | `??` used to hide missing values |
      | `try-catch-usage.test.js` | template | `try`/`catch` outside an allow-list that only shrinks |
      | `data-fallbacks.test.js` | template | `item.data.foo ?? default` on typed collection data |

      The template's fallback check states the philosophy in its header:

      ```javascript
      /**
       * The philosophy: throw errors instead of masking problems with
       * fallbacks. Defaults should be set early in the data chain
       * (collections, computed data), not scattered throughout filters
       * and utilities.
       *
       * BAD:
       *   const items = getItems() || [];
       *   const title = data.title || "Untitled";
       *
       * GOOD:
       *   const items = getItems();  // Throw if undefined
       *   const { title } = data;    // Trust the type system
       */
      ```

      ## Trust the invariants you own

      The other half of fail-loudly is not defending against states your
      own code makes impossible:

      > **Trust application invariants**: Do not design normal paths around
      > database states the application says are impossible. If an
      > impossible state is observed, raise it and repair the data
      > explicitly.

      So a state machine reader throws on a stored state word it does not
      know (`parseMachineState` in
      [`machine-spec.ts`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/schema-atlas/machine-spec.ts)),
      instead of mapping it to a default. The impossible state shows up in
      the log, and a separate live scan shows it to the operator so they can
      repair it.
---
