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

      ## The rest of the strict config

      The table shows the rules with the best stories. The tickets config
      turns on many more, all as errors. Most are small "one way to write
      it" rules. Each one alone is minor. Together they mean an agent's
      code looks like the rest of the codebase, so a reviewer reads
      meaning, not style.

      - **Complexity:** [`noStaticOnlyClass`](https://biomejs.dev/linter/rules/no-static-only-class/), [`noThisInStatic`](https://biomejs.dev/linter/rules/no-this-in-static/), [`noUselessStringConcat`](https://biomejs.dev/linter/rules/no-useless-string-concat/), [`noUselessTernary`](https://biomejs.dev/linter/rules/no-useless-ternary/), [`noUselessThisAlias`](https://biomejs.dev/linter/rules/no-useless-this-alias/), [`noUselessUndefined`](https://biomejs.dev/linter/rules/no-useless-undefined/), [`useArrowFunction`](https://biomejs.dev/linter/rules/use-arrow-function/), [`useDateNow`](https://biomejs.dev/linter/rules/use-date-now/), [`useFlatMap`](https://biomejs.dev/linter/rules/use-flat-map/), [`useLiteralKeys`](https://biomejs.dev/linter/rules/use-literal-keys/), [`useOptionalChain`](https://biomejs.dev/linter/rules/use-optional-chain/), [`useRegexLiterals`](https://biomejs.dev/linter/rules/use-regex-literals/), [`useWhile`](https://biomejs.dev/linter/rules/use-while/).
      - **Style:** [`noUnusedTemplateLiteral`](https://biomejs.dev/linter/rules/no-unused-template-literal/), [`noUselessElse`](https://biomejs.dev/linter/rules/no-useless-else/), [`noYodaExpression`](https://biomejs.dev/linter/rules/no-yoda-expression/), [`useAsConstAssertion`](https://biomejs.dev/linter/rules/use-as-const-assertion/), [`useCollapsedElseIf`](https://biomejs.dev/linter/rules/use-collapsed-else-if/), [`useConsistentArrayType`](https://biomejs.dev/linter/rules/use-consistent-array-type/), [`useConsistentArrowReturn`](https://biomejs.dev/linter/rules/use-consistent-arrow-return/), [`useExponentiationOperator`](https://biomejs.dev/linter/rules/use-exponentiation-operator/), [`useNumberNamespace`](https://biomejs.dev/linter/rules/use-number-namespace/), [`useShorthandAssign`](https://biomejs.dev/linter/rules/use-shorthand-assign/), [`useShorthandFunctionType`](https://biomejs.dev/linter/rules/use-shorthand-function-type/), [`useSingleVarDeclarator`](https://biomejs.dev/linter/rules/use-single-var-declarator/), [`useTrimStartEnd`](https://biomejs.dev/linter/rules/use-trim-start-end/).
      - **Suspicious:** [`noAssignInExpressions`](https://biomejs.dev/linter/rules/no-assign-in-expressions/), [`noImplicitAnyLet`](https://biomejs.dev/linter/rules/no-implicit-any-let/).
      - **Nursery** (new rules Biome has not finished yet): `useSortedClasses`, which keeps the class names in a `class` attribute in a fixed order, beside the file-length ceiling in the table.

      Three settings go beyond single rules:

      - `"domains": { "project": "all" }` turns on every rule in Biome's
        [project domain](https://biomejs.dev/linter/domains/), the rules
        that need to see more than one file at a time.
      - The [assist actions](https://biomejs.dev/assist/)
        `useSortedKeys`, `useSortedProperties`, `useSortedAttributes` and
        `useSortedInterfaceMembers` are on. Object keys, JSX attributes and
        interface members are in alphabetical order, so a diff never
        argues about order, and two agents editing the same object do not
        both append to the end.
      - Cognitive complexity is set to `warn`, not `error`, but the gate
        runs with `--error-on-warnings`. A warning fails CI the same as an
        error.

      ## Overrides by folder, not by line

      Strictness changes by folder, in `biome.json`, where a reviewer can
      see it. Production code under `src/` gets a complexity limit of 12,
      against 13 elsewhere. Tests may have unused variables and parameters
      and may log. Scripts may log. The logger itself may call `console`.
      There are no `biome-ignore` comments scattered through the code for
      an agent to copy.

      A few rules are off. `noUnresolvedImports` and
      `noUndeclaredDependencies` are off because Deno resolves imports
      through its own import map, which those rules do not read. The
      unresolved-import job moves to the type checker (see
      [Strict type checking](/techniques/static-checks/strict-types/)).

      ## The shared base config

      The chobble-template keeps its rules in
      [`packages/js-toolkit/configs/biome.base.json`](https://github.com/chobbledotcom/chobble-template/blob/main/packages/js-toolkit/configs/biome.base.json),
      and every project's `biome.json` extends it. One change to the base
      reaches every site. On top of the rules above, the base adds
      [`noParameterAssign`](https://biomejs.dev/linter/rules/no-parameter-assign/), [`noEmptyBlockStatements`](https://biomejs.dev/linter/rules/no-empty-block-statements/), [`useIsArray`](https://biomejs.dev/linter/rules/use-is-array/), [`noGlobalDirnameFilename`](https://biomejs.dev/linter/rules/no-global-dirname-filename/), [`useNodejsImportProtocol`](https://biomejs.dev/linter/rules/use-nodejs-import-protocol/),
      and the nursery rule [`noUnnecessaryConditions`](https://biomejs.dev/linter/rules/no-unnecessary-conditions/), which flags an
      `if` that can only ever go one way. Tests get Biome's `test` domain
      in full. The same `configs/` folder holds shared base configs for
      jscpd, Knip and Bun's test runner.

      Stylesheets have a linter too. The template's
      [`.stylelintrc.json`](https://github.com/chobbledotcom/chobble-template/blob/main/.stylelintrc.json)
      runs [Stylelint](https://stylelint.io/) with the standard SCSS rules,
      a fixed property order, and a class-name pattern that allows only
      kebab-case with optional
      [BEM](https://css-tricks.com/bem-101/) `__element` and `--modifier`
      parts.

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
