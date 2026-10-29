---
name: Strict type checking
subtitle: Every strict TypeScript flag on, and a ratchet where it cannot be on yet
guide-category: static-checks
order: 2
meta_description: >-
  The extra-strict TypeScript compiler flags the tickets repository uses,
  what each one catches, how 340 errors were fixed in one change, and the
  chobble-template's ratchet for moving JavaScript to strict mode file by
  file.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A [type checker](/glossary/#type-checker) reads your code and proves
      that values are used the way their types say, before anything runs.
      [TypeScript](https://www.typescriptlang.org/docs/) has a `strict`
      setting and several extra flags that are not part of `strict`. The
      [TSConfig reference](https://www.typescriptlang.org/tsconfig/)
      explains every flag on this page with a short example. Deno reads the
      same flags from `compilerOptions` in `deno.json`
      ([Deno docs](https://docs.deno.com/runtime/reference/ts_config_migration/)).
  - type: markdown
    content: |
      ## Types are a test suite that never needs writing

      An agent writes code that runs on the happy path. The type checker is
      where the unhappy paths show up first: the value that can be
      `undefined`, the `switch` that forgets a case, the function that
      returns nothing on one branch. Every flag you turn on is a whole class
      of bug the agent can no longer ship, checked in seconds.

      `"strict": true` is the start, not the end. TypeScript keeps several
      useful checks outside `strict`, because turning them on breaks
      existing code. In a codebase that agents maintain, that breakage is
      cheap to fix and worth fixing once.

  - type: split-code
    subtitle: tickets deno.json
    content: |
      ## The tickets compiler flags

      Everything in `strict`, plus every extra flag that finds bugs:

      - [`noUncheckedIndexedAccess`](https://www.typescriptlang.org/tsconfig/#noUncheckedIndexedAccess):
        `list[0]` has type `T | undefined`, not `T`. The empty list is no
        longer a surprise.
      - [`exactOptionalPropertyTypes`](https://www.typescriptlang.org/tsconfig/#exactOptionalPropertyTypes):
        a field marked `name?: string` may be missing, but may not be set
        to `undefined` unless the type says so.
      - [`noImplicitReturns`](https://www.typescriptlang.org/tsconfig/#noImplicitReturns):
        every branch of a function must return.
      - [`noFallthroughCasesInSwitch`](https://www.typescriptlang.org/tsconfig/#noFallthroughCasesInSwitch):
        no `case` runs on into the next by accident.
      - [`noImplicitOverride`](https://www.typescriptlang.org/tsconfig/#noImplicitOverride):
        a method that replaces a parent's method must say `override`.
      - [`noUnusedLocals`](https://www.typescriptlang.org/tsconfig/#noUnusedLocals)
        and [`noUnusedParameters`](https://www.typescriptlang.org/tsconfig/#noUnusedParameters):
        the type checker backs up the linter on dead names.
    figure_filename: deno.json
    figure_language: json
    figure_code: |
      "compilerOptions": {
        "strict": true,
        "noImplicitAny": true,
        "strictNullChecks": true,
        "noUncheckedIndexedAccess": true,
        "noImplicitReturns": true,
        "noFallthroughCasesInSwitch": true,
        "noImplicitOverride": true,
        "noUnusedLocals": true,
        "noUnusedParameters": true,
        "exactOptionalPropertyTypes": true
      }

  - type: markdown
    content: |
      ## Turning them on: 340 errors in one change

      Six of those flags arrived together in
      [#1471](https://github.com/chobbledotcom/tickets/pull/1471). The
      first commit turned them on and reported **343 type errors**. The
      branch was marked "NOT GREEN" in its commit messages, and several
      agents fixed groups of files in parallel, with a snapshot commit after
      each group so no work was lost. The final commit touched 129 files.

      Most of the errors came from `exactOptionalPropertyTypes`. The fix
      was to say what the code already did: an optional field that really
      does receive `undefined` became `?: T | undefined`. A handful of
      shared types, such as the HTML attribute type, fixed hundreds of
      errors at once.

      The pull request also shows when *not* to turn a flag on.
      `noPropertyAccessFromIndexSignature` was tried and dropped: it asked
      for 604 rewrites from `obj.key` to `obj["key"]`, and none of them
      found a bug. A flag earns its place by catching mistakes, not by
      being strict.

      ## The check is a gate

      [`deno task typecheck`](https://github.com/chobbledotcom/tickets/blob/main/deno.json)
      runs `deno check` over `src`, `test`, `cli` and `scripts`, and again
      over the separate payment test project with its own config. Tests,
      scripts and the command-line tools get the same strict flags as
      production code. The test runner does not type check, so AGENTS.md
      lists `deno task typecheck` among the gates every agent runs before
      it commits (see
      [Precommit](/techniques/workflow/precommit/)).

  - type: split-code
    subtitle: chobble-template
    content: |
      ## When you cannot turn it on yet: a ratchet

      The chobble-template is JavaScript checked by TypeScript through
      JSDoc comments, with `checkJs` on. Its normal
      `tsconfig.json` is not strict, because the first strict run found
      631 errors.

      Instead of fixing all of them at once, the template added
      [`tsconfig.strict.json`](https://github.com/chobbledotcom/chobble-template/blob/main/tsconfig.strict.json),
      which is the normal config with `strict: true`, and a
      [ratchet script](https://github.com/chobbledotcom/chobble-template/blob/main/scripts/strict-typecheck-ratchet.js)
      ([#1088](https://github.com/chobbledotcom/chobble-template/pull/1088))
      that runs it with two rules:

      1. The total number of strict errors must not go above the recorded
         count. That count started at 631 and is now 343.
      2. A file on the "strict-clean" list must stay at zero errors. The
         list grows as files get fixed. It started with 38 files and holds
         94 now.

      `bun run typecheck:strict` runs in precommit. New code in a clean
      file is held to full strict mode from day one, and the old errors
      can only go down. See [Ratchets](/techniques/static-checks/ratchets/)
      for the general pattern.
    figure_filename: scripts/strict-typecheck-ratchet.js
    figure_language: javascript
    figure_code: |
      // Current baseline - lower this as you fix errors
      const CURRENT_ERROR_COUNT = 343;

      // Files that currently pass strict mode
      // (must not regress)
      const STRICT_CLEAN_FILES = [
        "packages/js-toolkit/fp/index.js",
        "src/_data/config.js",
        // …
      ];

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      Turn on `strict`, then add `noUncheckedIndexedAccess` and
      `exactOptionalPropertyTypes`. Those two find the most real bugs. If
      the error count is too big for one change, record it and make it
      a number that can only go down, and keep a list of files that are
      already clean.
---
