---
name: A strict linter
subtitle: Biome with the opinionated rules switched on, run read-only in CI
guide-category: static-checks
order: 1
meta_description: >-
  The Biome configuration behind the tickets and chobble-template
  repositories: complexity limits, no forEach, no accumulating spreads,
  strict equality, no focused or skipped tests, and a read-only CI lint.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Lint is the cheapest teacher

      A linter tells an agent it is wrong within a second, with the line
      number, every time. That makes it the best place to encode style
      rules: an agent learns the house style from lint errors far faster
      than from prose.

      Both repositories use [Biome](https://biomejs.dev/), pinned to an
      exact version, with many rules raised from "suggestion" to "error".

      ## Rules worth switching on

      From the tickets
      [`biome.json`](https://github.com/chobbledotcom/tickets/blob/main/biome.json):

      | Rule | Why it matters with agents |
      | --- | --- |
      | `noExcessiveCognitiveComplexity` (12 in `src/`) | Stops the 80-line function with six nested branches. Agents grow functions instead of splitting them. |
      | `noForEach` | Forces `for...of` or the curried `#fp` helpers. |
      | `noAccumulatingSpread` | `[...acc, item]` inside a reduce is O(n²). Agents write it constantly. |
      | `noDoubleEquals` with `ignoreNull: false` | No loose equality, not even for `null`. This also kills a whole class of mutants before tests run. |
      | `noUnusedImports`, `noUnusedVariables`, `noUnusedFunctionParameters` | Debris from refactors does not survive. |
      | `noConsole` | Logging goes through the logger. Scripts are the exception. |
      | `noFocusedTests`, `noSkippedTests` | An agent cannot `.only` or `.skip` its way to a green run. |
      | `noExcessiveLinesPerFile` (1,000) | A hard ceiling under the softer file-length ratchet. |
      | `useThrowNewError`, `useConst`, `noVar`, `useTemplate` | One way to write each common thing. |

      The [chobble-template](https://github.com/chobbledotcom/chobble-template)
      goes further on complexity: its shared base config sets
      `maxAllowedComplexity` to **7** as an error, with 30 allowed in tests.
      Seven is low. It forces small functions composed with `pipe`, which is
      what the codebase wants anyway.

      ## Two tasks: one to fix, one to judge

      The tickets repository splits linting into two tasks:

      - `deno task lint` runs `biome check --write --unsafe`. Formatting
        and fixable findings, including unsafe fixes like removing unused
        imports, heal themselves.
      - `deno task lint:ci` runs `biome check --error-on-warnings` and
        changes nothing. This is the gate in precommit and CI, so local
        precommit is exactly as strict as CI and never edits the checkout
        while it is judging it.

      A manual GitHub workflow, `biome-fix.yml`, runs the fixing task on a
      chosen branch and opens a pull request with the result, for the
      occasional branch that lands badly formatted.

      ## Lint as a mutation filter

      The [mutation tester](/techniques/testing/mutation-testing/) runs
      Biome on every mutant before running any tests. A mutant that turns
      `===` into `==` is rejected by `noDoubleEquals` and counts as killed
      for free. A strict lint config therefore makes mutation testing
      faster, and it means the list of known-equivalent mutants never needs
      entries for loose equality.

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      Pin your linter version. Turn the warnings you care about into
      errors. Run a read-only check in CI with warnings as errors, and keep
      a separate task that applies fixes. Set cognitive complexity lower
      than feels comfortable.
---
