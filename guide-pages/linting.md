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
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A [linter](/glossary/#linter) reads your code without running it and points at
      likely mistakes. This page uses [Biome](https://biomejs.dev/guides/getting-started/).
      Every rule name below links to Biome's page for that rule, which shows a
      bad and a good example. If you have used [ESLint](https://eslint.org/), the
      ideas are the same.
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
      | [`noExcessiveCognitiveComplexity`](https://biomejs.dev/linter/rules/no-excessive-cognitive-complexity/) (12 in `src/`) | Stops the 80-line function with six nested branches. Agents grow functions instead of splitting them. |
      | [`noForEach`](https://biomejs.dev/linter/rules/no-for-each/) | Forces `for...of` or the curried `#fp` helpers. |
      | [`noAccumulatingSpread`](https://biomejs.dev/linter/rules/no-accumulating-spread/) | `[...acc, item]` inside a reduce is O(n²). Agents write it constantly. |
      | [`noDoubleEquals`](https://biomejs.dev/linter/rules/no-double-equals/) with `ignoreNull: false` | No loose equality, not even for `null`. This also kills a whole class of mutants before tests run. |
      | [`noUnusedImports`](https://biomejs.dev/linter/rules/no-unused-imports/), [`noUnusedVariables`](https://biomejs.dev/linter/rules/no-unused-variables/), [`noUnusedFunctionParameters`](https://biomejs.dev/linter/rules/no-unused-function-parameters/) | Debris from refactors does not survive. |
      | [`noConsole`](https://biomejs.dev/linter/rules/no-console/) | Logging goes through the logger. Scripts are the exception. |
      | [`noFocusedTests`](https://biomejs.dev/linter/rules/no-focused-tests/), [`noSkippedTests`](https://biomejs.dev/linter/rules/no-skipped-tests/) | An agent cannot `.only` or `.skip` its way to a green run. |
      | [`noExcessiveLinesPerFile`](https://biomejs.dev/linter/rules/no-excessive-lines-per-file/) (1,000) | A hard ceiling under the softer file-length ratchet. |
      | [`useThrowNewError`](https://biomejs.dev/linter/rules/use-throw-new-error/), [`useConst`](https://biomejs.dev/linter/rules/use-const/), [`noVar`](https://biomejs.dev/linter/rules/no-var/), [`useTemplate`](https://biomejs.dev/linter/rules/use-template/) | One way to write each common thing. |

      The [chobble-template](https://github.com/chobbledotcom/chobble-template)
      goes further on complexity: its shared base config sets
      `maxAllowedComplexity` to **7** as an error, with 30 allowed in tests.
      Seven is low. It forces small functions composed with `pipe`, which is
      what the codebase wants anyway.

      ## Two tasks: one to fix, one to judge

      The tickets repository splits linting into two tasks:

      - [`deno task lint`](https://github.com/chobbledotcom/tickets/blob/main/scripts/biome.ts) runs `biome check --write --unsafe`. Formatting
        and fixable findings, including unsafe fixes like removing unused
        imports, heal themselves.
      - [`deno task lint:ci`](https://github.com/chobbledotcom/tickets/blob/main/scripts/biome.ts) runs `biome check --error-on-warnings` and
        changes nothing. This is the gate in precommit and CI, so local
        precommit is exactly as strict as CI and never edits the checkout
        while it is judging it.

      A manual GitHub workflow, [`biome-fix.yml`](https://github.com/chobbledotcom/tickets/blob/main/.github/workflows/biome-fix.yml), runs the fixing task on a
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
