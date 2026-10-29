---
name: Code-quality tests
subtitle: Tests that read the source code and fail on patterns you have banned
guide-category: static-checks
order: 7
meta_description: >-
  How the chobble-template uses ordinary unit tests to scan its own source
  for banned patterns: single-use functions, mutable consts, html in
  JavaScript, test-only exports, TODO comments and more.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      These are ordinary [unit tests](/glossary/#unit-test), run by
      [Bun's test runner](https://bun.sh/docs/test/writing), except that the
      code they test is the project's own source files. Most use a
      [regular expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions)
      to find a banned pattern. Every check name in the table links to its
      test file.
  - type: markdown
    content: |
      ## When the linter does not have the rule you want

      [Biome](https://biomejs.dev/guides/getting-started/) and [ESLint](https://eslint.org/) cover the common rules. Every codebase also has its
      own: "don't call `process.cwd()` in tests", "don't put HTML in
      JavaScript strings", "don't mutate objects". Writing a linter plugin
      for each is heavy. Writing a test is not.

      The [chobble-template](https://github.com/chobbledotcom/chobble-template)
      keeps around forty of these in
      [`test/unit/code-quality/`](https://github.com/chobbledotcom/chobble-template/tree/main/test/unit/code-quality).
      Each one reads the source files, looks for a pattern, and fails with
      the file and line of every violation. They run with the rest of the
      suite, so they need no extra tooling.

      ## A sample of what they catch

      | Test | What it bans |
      | --- | --- |
      | [`single-use-functions`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/single-use-functions.test.js) | Unexported functions called exactly once, which add indirection without reuse |
      | [`test-only-exports`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/test-only-exports.test.js) | Exports from `src/` that only tests import |
      | [`object-mutation`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/object-mutation.test.js) | Mutating objects in place instead of building new ones |
      | [`let-usage`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/let-usage.test.js) | `let` where `const` and a pipeline would do |
      | [`html-in-js`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/html-in-js.test.js) | HTML built in JavaScript strings instead of templates |
      | [`or-fallbacks`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/or-fallbacks.test.js), [`nullish-coalescing`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/nullish-coalescing.test.js) | Defaults that hide missing data |
      | [`try-catch-usage`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/try-catch-usage.test.js) | `try`/`catch` outside a short allow-list |
      | [`commented-code`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/commented-code.test.js) | Code left in comments |
      | [`todo-fixme-comments`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/todo-fixme-comments.test.js) | `TODO` and `FIXME`, which belong in issues |
      | [`memoize-inside-function`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/memoize-inside-function.test.js) | A cache created per call, which caches nothing |
      | [`relative-paths`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/relative-paths.test.js) | `../../` imports where an alias exists |
      | [`unused-classes`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/unused-classes.test.js), [`unused-images`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/unused-images.test.js) | CSS classes and image files nothing uses |
      | [`duplicate-methods`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/duplicate-methods.test.js) | Two methods with the same body |
      | [`backwards-compat`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/backwards-compat.test.js) | Compatibility shims for internal APIs |
      | `lockfile` | A lockfile from the wrong package manager |
      | [`pages-yml-freshness`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/pages-yml-freshness.test.js) | A generated CMS config that has drifted from its source |

      ## The shape of one check

      Each test file starts with a comment that says what it detects and
      why, then uses a shared scanner from the toolkit package. The header
      of [`single-use-functions.test.js`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/single-use-functions.test.js) is typical:

      ```javascript
      /**
       * Detects unexported functions that are only called once.
       *
       * Single-use unexported functions are candidates for inlining into
       * their caller, as they add indirection without reuse benefit. This
       * is a warning, not an error, since some single-use functions are
       * intentionally named for clarity.
       */
      ```

      The scanner ([`packages/js-toolkit/code-quality/`](https://github.com/chobbledotcom/chobble-template/tree/main/packages/js-toolkit/code-quality)) handles reading
      files, matching patterns, skipping comments, and reporting. A new
      check is mostly a regular expression and an explanation.

      ## Exceptions live in one file, and only shrink

      Every grandfathered violation lives in one file,
      [`test/code-quality/code-quality-exceptions.js`](https://github.com/chobbledotcom/chobble-template/blob/main/test/code-quality/code-quality-exceptions.js),
      as `file:line` or whole-file entries with a comment explaining each
      one. The file opens with a banner telling every reader, human or
      agent, that the only valid change is a deletion. Several checks also
      fail on a *stale* exception, one that no longer matches a violation,
      so fixed code cannot leave a loophole behind for the next change.

      ## Checks that check the checks

      The tickets repository takes this one step further: its precommit
      tooling is itself under test in [`test/scripts/`](https://github.com/chobbledotcom/tickets/tree/main/test/scripts), with 100% coverage.
      A check that silently stops matching is worse than no check, because
      everyone believes it is still working.

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      Next time you write a review comment that says "we don't do that
      here", write a test instead. Put a paragraph at the top saying why.
      Put the existing violations in one exceptions file that only shrinks,
      and make the test fail on stale exceptions.
---
