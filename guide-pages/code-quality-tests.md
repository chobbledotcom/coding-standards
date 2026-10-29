---
name: Code-quality tests
subtitle: Tests that read the source code and fail on patterns you have banned
guide-category: static-checks
order: 8
meta_description: >-
  The complete list of code-quality tests in the chobble-template and the
  tickets repository: ordinary unit tests that scan the project's own
  source for banned patterns.
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
      keeps 42 of these in
      [`test/unit/code-quality/`](https://github.com/chobbledotcom/chobble-template/tree/main/test/unit/code-quality).
      Each one reads the source files, looks for a pattern, and fails with
      the file and line of every violation. They run with the rest of the
      suite, so they need no extra tooling.

      ## Every check, grouped

      All 42 tests in the folder, grouped by what they protect. Each name
      links to the test file, which explains the rule in its header.

      ### Hiding errors

      | Test | What it bans |
      | --- | --- |
      | [`or-fallbacks`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/or-fallbacks.test.js) | `x \|\| []`, `x \|\| ""` and other `\|\|` defaults that hide missing data. Defaults belong early in the data chain. |
      | [`nullish-coalescing`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/nullish-coalescing.test.js) | `??` anywhere outside the collection builders and shared packages, where defaults are set once |
      | [`data-fallbacks`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/data-fallbacks.test.js) | `item.data.title ?? ""` style fallbacks on collection data, which is already typed |
      | [`try-catch-usage`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/try-catch-usage.test.js) | `try`/`catch` outside the allow-list in the exceptions file |

      ### Mutation and state

      | Test | What it bans |
      | --- | --- |
      | [`let-usage`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/let-usage.test.js) | `let`, and "mutable `const`": `const x = []`, `{}`, `new Set()` or `new Map()` |
      | [`array-push`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/array-push.test.js) | `.push()`. Build a new array with `map`, `filter` or `reduce`. |
      | [`object-mutation`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/object-mutation.test.js) | `obj[key] = value` assignments |

      ### Size and structure

      | Test | What it bans |
      | --- | --- |
      | [`function-length`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/function-length.test.js) | Functions over 30 lines (20 preferred) |
      | [`single-use-functions`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/single-use-functions.test.js) | Unexported functions called exactly once, which add indirection without reuse |
      | [`duplicate-methods`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/duplicate-methods.test.js) | The same function name defined in more than one file, a sign that two files should share one helper |
      | [`test-only-exports`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/test-only-exports.test.js) | Exports from `src/` and `packages/` that only tests import |
      | [`memoize-inside-function`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/memoize-inside-function.test.js) | `memoize()` inside a function body, which makes a new, empty cache on every call |
      | [`nested-array-lookup`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/nested-array-lookup.test.js) | `.find()` or `.filter()` inside another loop's callback: O(n×m) work where an index would be O(n+m) |
      | [`short-circuit-order`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/short-circuit-order.test.js) | `arr.includes(x) && id === y`. Put the cheap equality check first. |

      ### Naming, style and comments

      | Test | What it bans |
      | --- | --- |
      | [`aliasing`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/aliasing.test.js) | `const alias = fn`, `const alias = obj.prop`: a second name for the same thing |
      | [`destructuring`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/destructuring.test.js) | `const { foo } = bar`. Write `bar.foo` so the source of each value stays visible. |
      | [`naming-conventions`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/naming-conventions.test.js) | camelCase names longer than four words |
      | [`then-usage`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/then-usage.test.js) | `.then()` chains. Use `async`/`await`. |
      | [`relative-paths`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/relative-paths.test.js) | `../` imports, and over-broad `#src/` aliases, where a specific alias exists |
      | [`inline-type-annotations`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/inline-type-annotations.test.js) | JSDoc `@type` comments inside function bodies |
      | [`comment-limits`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/comment-limits.test.js) | More than five lines of inline comments in a file, beyond the header. Agents over-comment. |
      | [`commented-code`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/commented-code.test.js) | Code left in comments |
      | [`todo-fixme-comments`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/todo-fixme-comments.test.js) | `TODO` and `FIXME`, which belong in issues |
      | [`backwards-compat`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/backwards-compat.test.js) | Any mention of "backwards compatibility", which in an internal codebase means a shim nobody needs |

      ### Templates, CSS and the site

      | Test | What it bans |
      | --- | --- |
      | [`html-in-js`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/html-in-js.test.js) | HTML built in JavaScript strings instead of templates |
      | [`url-construction`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/url-construction.test.js) | Hard-coded collection paths such as `/products/` in URLs. The paths come from the strings file, so a site can rename them. |
      | [`unused-classes`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/unused-classes.test.js) | Classes in the HTML that no stylesheet or script uses |
      | [`unused-images`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/unused-images.test.js) | Image files that no page uses |
      | [`design-system-scoping`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/design-system-scoping.test.js) | SCSS rules outside the `.design-system { }` scope |
      | [`template-selectors`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/template-selectors.test.js) | An element ID in `selectors.js` that no HTML template contains |
      | [`unregistered-collections`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/unregistered-collections.test.js) | `collections.foo` in a template when no `foo` collection is registered |
      | [`data-exports`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/data-exports.test.js) | Data files that mix named and default exports, which breaks Eleventy |
      | [`block-markdown-rendering`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/block-markdown-rendering.test.js) | A content block that renders a plain-text field as Markdown, or a Markdown field without the `prose` styles |

      ### Generated files

      | Test | What it bans |
      | --- | --- |
      | [`pages-yml-freshness`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/pages-yml-freshness.test.js) | A committed CMS config, `.pages.yml`, that no longer matches what its generator produces |
      | [`pages-yml-reference-names`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/pages-yml-reference-names.test.js) | A collection entry the CMS refers to that has no `name` field |
      | [`type-generation-freshness`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/type-generation-freshness.test.js) | Generated TypeScript types that have drifted from `.pages.yml` |
      | [`lockfile`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/lockfile.test.js) | Any lockfile except `bun.lock` |

      ### Tests

      | Test | What it bans |
      | --- | --- |
      | [`test-quality`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/test-quality.test.js) | Vague test names (`test1`, `works`), names with more than one "and", assertions without a message, set-then-assert tautologies, and `async` tests that never `await` |
      | [`mock-module-usage`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/mock-module-usage.test.js) | Bare `mock.module()`, which leaks into other test files. Use the `mockModule` helper, which restores the module. |
      | [`dom-mocking`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/dom-mocking.test.js) | `globalThis.document` and `new DOM()`, when the test DOM is already global |

      ### The checkers themselves

      | Test | What it bans |
      | --- | --- |
      | [`code-scanner`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/code-scanner.test.js) | Unit tests for the shared scanner that every check above uses |
      | [`test-hygiene`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/test-hygiene.test.js) | Empty file lists. A check that scans no files passes, and nobody notices. |

      ## The same idea in tickets

      The tickets repository keeps its version in one file,
      [`test/integration/code-quality.test.ts`](https://github.com/chobbledotcom/tickets/blob/main/test/integration/code-quality.test.ts).
      It scans the real `src/` and `test/` trees and expects zero
      violations. The detectors live in
      [`test/scripts/code-quality/`](https://github.com/chobbledotcom/tickets/tree/main/test/scripts/code-quality),
      and each one has its own fixture tests that prove it fires on bad
      code and stays quiet on good code. These are all ten rules:

      | Rule | What it bans |
      | --- | --- |
      | No in-memory state | Module-level `Map` or `Set` in `src/`. An edge function can run on many isolates, so state there is not shared. Six caches are allowed, each with a comment. |
      | DB writes go through the client | `getDb().execute` or `.batch` outside the database client. A raw write skips cache invalidation. |
      | Numbered parameters | SQL text that spells a `?N` token by hand. Each value binds once through the numbered binder. |
      | No aliasing | A module-level second name for an existing function or value |
      | No module-level `let` | Mutable module state. Use `const` with `once()` or `lazyRef()`. |
      | No `.then()` | Promise chains. Use `async`/`await`. |
      | No `../` imports | Relative parent imports. Use a `#` alias. |
      | No test-only exports | Exports from `src/` that only tests use. See [Dead code](/techniques/static-checks/dead-code/). |
      | No redundant constant arguments | A function that every caller passes the same constant. Use a default parameter or a constant instead. |
      | No duplicate type shapes | Two differently named types with the same members. Make them one shared type. |

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
