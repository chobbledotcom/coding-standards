---
name: Ratchets
subtitle: Lists that only shrink, so a check is strict for new code on day one
guide-category: static-checks
order: 3
meta_description: >-
  The ratchet pattern: baselines, registries and exception lists that can
  only get smaller, with update commands that refuse a rise. Used for
  coverage exclusions, file lengths, casts, assertions, prose and more.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A [ratchet](/glossary/#ratchet) is a list of known problems that may only get
      shorter. The name comes from the
      [mechanical ratchet](https://en.wikipedia.org/wiki/Ratchet_(device)),
      which turns one way and locks against the other. You will see the same
      idea called a *baseline* in many tools, for example
      [Biome's suppressions](https://biomejs.dev/linter/#suppress-lint-rules)
      or [Detekt baselines](https://detekt.dev/docs/introduction/baseline/).
  - type: markdown
    content: |
      ## The problem every new check has

      You write a new check. You run it. It finds four hundred problems in
      existing code. You now have three bad choices: fix all four hundred
      before you can merge the check, switch it on as a warning that
      everyone ignores, or exclude the old files and hope nobody adds a new
      one to the list.

      A ratchet is the fourth choice. The check records today's findings in
      a committed baseline. New code must be clean. Old code can only get
      cleaner. The baseline can go down and can never go up.

      ## Ratchets in the tickets repository

      Nearly every check that started on a dirty codebase is a ratchet:

      | Ratchet | What it holds |
      | --- | --- |
      | [`check-coverage-exclusions/exclusions.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-coverage-exclusions/exclusions.ts) | Files the coverage gate skips. A branch may not add one. |
      | [`check-file-lengths/over-limit.json`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-file-lengths/over-limit.json) | Files over 500 lines, with their current length. They may shrink, never grow. |
      | [`check-double-casts/baseline.json`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-double-casts/baseline.json) | Counts of `as unknown as` per file. |
      | [`check-presence-assertions/registry.json`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-presence-assertions/registry.json) | Counts of `toBeDefined()` per test file. |
      | [`check-ste/baseline.json`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-ste/baseline.json) | STE findings per Markdown document. |
      | [`check-comments/ste-baseline.json`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-comments/ste-baseline.json) | STE findings per source file's comments. |
      | [`unread-fields/baseline.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/unread-fields/baseline.ts) | Exported fields that nothing reads. |
      | [`.jscpd.renamed-baseline.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.renamed-baseline.json) | Accepted near-identical clones, by fingerprint. |
      | [`test/integration/import-cycles.test.ts`](https://github.com/chobbledotcom/tickets/blob/main/test/integration/import-cycles.test.ts) | Import rings in `src/`. Once it reached zero, the cycle report became a hard gate. |

  - type: split-code
    subtitle: Example
    content: |
      ## The file-length baseline

      The limit is 500 lines. These files were already over it when the
      check arrived. Each number is the most lines that file may have. When
      someone splits a file, `--update` lowers or removes its entry, and
      the update refuses to raise one.
    figure_filename: scripts/check-file-lengths/over-limit.json
    figure_language: json
    figure_code: |
      {
        "src/features/auth.ts": 727,
        "src/fp.ts": 585,
        "src/shared/db/attendees/servicing.ts": 905,
        "src/shared/db/client.ts": 915,
        "src/shared/db/groups.ts": 800,
        "src/shared/db/table.ts": 764,
        "src/shared/limits.ts": 674,
        ...
      }

  - type: markdown
    content: |
      ## What makes a ratchet hold

      A baseline file on its own is just another list an agent can edit. The
      ratchets in tickets have three properties that make them hold:

      1. **The update command refuses a rise.** `--update` re-records the
         counts after a fix, and fails if any count went up. From the
         double-cast checker: "The update refuses a baseline that rose, so a
         rise must be removed first. A removed entry can no longer be
         recorded anew."
      2. **The check compares against the merge base.** The coverage
         exclusion and test-quality gates diff the branch against
         `origin/main`, so the question is "did this branch add one?", not
         "is the list long?".
      3. **The rule says the list only shrinks.** AGENTS.md says it in the
         heading ("**Never add a coverage exclusion**"), and the file says it
         again at the top:

      > The list only ever shrinks: a branch must not add an entry, because
      > an exclusion hides a coverage gap instead of fixing it. A reader who
      > needs to answer a gap restructures the code so the coverage merge
      > reads it.

      ## The chobble-template versions

      The template uses the same idea with different mechanics:

      - **[`cpd:ratchet`](https://github.com/chobbledotcom/chobble-template/blob/main/scripts/cpd-ratchet.js)** reads the current `--min-tokens` from
        `package.json`, then runs jscpd at one token lower. If the codebase
        would pass at the stricter setting, the check *fails*, forcing
        someone to tighten the threshold. The duplication limit can only go
        down.
      - **[`typecheck:strict`](https://github.com/chobbledotcom/chobble-template/blob/main/scripts/strict-typecheck-ratchet.js)** runs `tsc --strict` across the project. The
        total error count may not exceed a baseline, and a list of
        strict-clean files may not gain any errors.
      - **[`code-quality-exceptions.js`](https://github.com/chobbledotcom/chobble-template/blob/main/test/code-quality/code-quality-exceptions.js)** holds the grandfathered findings
        of the [code-quality tests](/techniques/static-checks/code-quality-tests/),
        under a banner that leaves no room for interpretation:

      ```text
      DO NOT ADD NEW ENTRIES TO THIS FILE UNDER ANY CIRCUMSTANCES.

      The ONLY valid changes to this file are DELETIONS.

      If your new code triggers a quality check failure:
        1. Fix the code to meet quality standards - no exceptions
        2. If you believe the check is wrong, fix the check itself
        3. There is no option 3 - adding exceptions is not allowed
      ```

      ## Why this matters more with agents

      An agent asked to make a failing check pass will look for the fastest
      route. The fastest route is almost always the exception list. A
      ratchet closes that route mechanically, and the written rule ("Never
      loosen a check to close a finding") closes it socially. Either alone
      leaks.

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      When you add a check, record the current findings in a committed file
      keyed by path. Make the check fail on any count above the baseline,
      and make the update command refuse to record a rise. Fail when a
      baseline entry no longer matches anything, so stale entries cannot
      hide future findings.
---
