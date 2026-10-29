---
name: The precommit pipeline
subtitle: Twenty-five gates in a fixed order, and a light set for shared machines
guide-category: workflow
order: 1
meta_description: >-
  The tickets precommit pipeline: twenty-five ordered gates from lint to
  model checking to full coverage, defined as data, recorded as status
  files, with a light gate set for agents on a shared machine.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A [pre-commit hook](/glossary/#git-hook) is a script git runs before each commit.
      The tickets hook runs `deno task precommit`. A `deno task` is a named
      command defined in `deno.json`, much like an npm script
      ([Deno task docs](https://docs.deno.com/runtime/reference/cli/task/)).
      Every step in the table links to the script that runs it.
  - type: markdown
    content: |
      ## One command, every check

      [`deno task precommit`](https://github.com/chobbledotcom/tickets/blob/main/scripts/precommit.ts) runs every gate the repository has, in order,
      and stops at the first failure. The git pre-commit hook runs it too,
      so nothing reaches a branch without passing it, unless the commit
      says why (see below).

      The steps are declared as data in
      [`scripts/precommit/steps.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/precommit/steps.ts),
      each with a comment saying which rule in AGENTS.md it enforces. That
      file is the best single index of the techniques on this site.

      | # | Step | What it holds |
      | --- | --- | --- |
      | 1 | `lint` | Biome, read-only, warnings as errors |
      | 2 | [`typecheck`](https://github.com/chobbledotcom/tickets/blob/main/deno.json) | `deno check` over every tree |
      | 3 | [`check:unread-fields`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-unread-fields.ts) | Exported fields nothing reads |
      | 4 | [`cpd`](https://github.com/chobbledotcom/tickets/blob/main/scripts/cpd.ts) | Eight duplication scans at 0% |
      | 5 | [`check:copy`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-copy.ts) | Simple language in the user copy |
      | 6 | [`check:comments`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-comments.ts) | 12 lines, 100 columns |
      | 7 | [`check:comment-ste`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-comments-ste.ts) | STE in comments |
      | 8 | [`check:imports`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-imports.ts) | One name per module |
      | 9 | [`check:effects`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-effects.ts) | The pure core reaches no IO |
      | 10 | [`check:alias-exports`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-alias-exports.ts) | No renaming re-exports |
      | 11 | [`check:empty-catch`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-empty-catch.ts) | Every catch recovers, re-raises or explains |
      | 12 | [`check:double-casts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-double-casts.ts) | `as unknown as`, against a baseline |
      | 13 | [`check:file-lengths`](https://github.com/chobbledotcom/tickets/blob/main/scripts/counts-check.ts) | 500 lines, against a baseline |
      | 14 | [`check:coverage-exclusions`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-coverage-exclusions.ts) | The exclusion list did not grow |
      | 15 | [`check:test-quality`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-test-quality.ts) | No weak assertion in a touched test file |
      | 16 | [`check:presence-assertions`](https://github.com/chobbledotcom/tickets/blob/main/scripts/counts-check.ts) | No new `toBeDefined()` |
      | 17 | [`contract:diff`](https://github.com/chobbledotcom/tickets/blob/main/scripts/contract-drift.ts) | Machine drift report runs on the base tree |
      | 18 | [`check:ste`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-ste.ts) | STE in Markdown |
      | 19 | [`check:e2e-labels`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-e2e-labels.ts) | Nightly e2e labels exist in the catalog |
      | 20 | [`check:equivalents`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-equivalent-mutants.ts) | Equivalent-mutant records still point at real mutants |
      | 21 | `cycles` | Zero import cycles |
      | 22 | [`check:required-checks`](https://github.com/chobbledotcom/tickets/blob/main/scripts/required-checks/check.ts) | GitHub's required checks match the manifest |
      | 23 | [`verify:models`](https://github.com/chobbledotcom/tickets/blob/main/scripts/verify-models.ts) | Apalache proofs, plus the counterexample self-test |
      | 24 | [`build:edge`](https://github.com/chobbledotcom/tickets/blob/main/scripts/build-edge.ts) | The production bundle builds |
      | 25 | [`test:coverage`](https://github.com/chobbledotcom/tickets/blob/main/scripts/run-tests.ts) | The full suite at 100% line and branch coverage |

      ## Cheapest first

      The order is deliberate. Lint and typecheck take seconds and catch the
      most. The text checks are next. Model checking and the edge build take
      longer. The full suite with coverage is last, because it is by far the
      slowest. A failure early in the list costs seconds, not minutes.

      ## The light gates

      Several agents share one machine. If every agent ran the full suite
      with coverage before every commit, the machine would spend its day
      running the same tests in parallel. So AGENTS.md defines a lighter
      set:

      > Run the tests for the files you changed with [`deno task test:files`](https://github.com/chobbledotcom/tickets/blob/main/scripts/run-test-files.ts),
      > then run these gates: [`lint:ci`](https://github.com/chobbledotcom/tickets/blob/main/scripts/biome.ts), `cpd`, `check:file-lengths`,
      > `check:imports`, `check:comment-ste`, `typecheck`, and `check:ste`
      > whenever the change touches Markdown.

      After those pass, the agent commits with `--no-verify` and puts
      `light gates passed; full precommit left to hosted CI` in the commit
      message. CI then runs everything. The phrase in the message is the
      important part: it makes the shortcut visible and searchable, rather
      than something an agent does quietly.

      ## Status records for supervisors

      When an agent asks "is precommit still running, or did it hang?", the
      answer should not need a guess. Each precommit run writes a small
      JSON record under `.diagnostics/precommit/`: the process id, the state
      (`waiting`, `running`, `passed`, `failed`), the current step and the
      exit code. Records are replaced atomically, hold no arguments, paths
      or log output, and are validated by a schema
      ([docs/precommit-status.md](https://github.com/chobbledotcom/tickets/blob/main/docs/precommit-status.md)).
      A separate read-only `agent-diagnose` tool reads them, so supervising
      agents can check on each other without reading logs.

      ## The branch mutation gate

      Mutation testing is too slow for every commit, so it has its own
      task, [`deno task precommit:mutation`](https://github.com/chobbledotcom/tickets/blob/main/scripts/precommit-mutation.ts), run once a branch is stable. It
      only mutates files the branch changed, and it works on the committed
      diff, so "commit any final source changes before running" it. See
      [Mutation testing](/techniques/testing/mutation-testing/).
---
