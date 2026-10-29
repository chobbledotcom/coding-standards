---
name: CI, stacks and merging
subtitle: CI that mirrors precommit, a manifest for required checks, and short stacks
guide-category: workflow
order: 2
meta_description: >-
  How changes reach main: CI jobs that run the same tasks as precommit, a
  reviewable manifest of required status checks, trial merges before a
  pull request is ready, and stacks of three to seven small pull requests.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      [Continuous integration](/glossary/#ci) runs your checks on a server for every
      push. The repositories use [GitHub Actions](https://docs.github.com/en/actions/get-started/understand-github-actions).
      *Required status checks* are the CI jobs GitHub insists pass before a
      merge ([GitHub docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches#require-status-checks-before-merging)).
      [Stacked pull requests](/glossary/#stacked-prs) are explained at
      [stacking.dev](https://www.stacking.dev/).
  - type: markdown
    content: |
      ## CI runs the same commands

      The `Test` workflow in tickets has four jobs: `checks`, `test`,
      `browser-contracts` and `specs`. The `checks` job is the precommit
      list, one named step per gate, each running the same `deno task` as
      local precommit inside the same [devenv](https://devenv.sh/getting-started/) profile. A failure in CI can
      always be reproduced locally with the command on the failing step.

      There is no separate "CI version" of a check. When CI and local
      disagree, the cause is the environment, and the environment is
      [pinned](/techniques/workflow/environment/).

      ## The gate itself is reviewable

      GitHub's required status checks live in repository settings, where no
      pull request can see or change them. That makes them easy to weaken
      by accident: a renamed job silently stops being required.

      The repository keeps a manifest,
      [`scripts/required-checks/checks.json`](https://github.com/chobbledotcom/tickets/blob/main/scripts/required-checks/checks.json):

      ```json
      ["checks", "specs", "test"]
      ```

      [`deno task check:required-checks`](https://github.com/chobbledotcom/tickets/blob/main/scripts/required-checks/check.ts) reads the live ruleset with `gh` and
      fails when it does not match the manifest. Changing the gate now
      shows up in a diff.

      ## Trial-merge before "ready"

      A pull request can pass CI on its own branch and still break `main`
      when merged, because `main` moved. [`deno task check:merged-tree`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-merged-tree.ts)
      merges the head into the base in a throwaway worktree, runs typecheck
      and the suite on the merged tree, and removes the worktree. A pull
      request is presented as ready only when both the head CI and this
      check pass.

      ## Stacked pull requests

      Large features land as a stack: a chain of small pull requests where
      each targets the one below it, managed with the `gh stack`
      extension. The rules in AGENTS.md:

      - Keep a stack at three to seven pull requests. Split a larger job
        into several stacks.
      - **Every layer must be independently green, reviewable, and useful
        on its own.** Delete the implementation a layer replaces in that
        same layer.
      - Review and merge from the bottom up, and rebase the stack after a
        lower layer changes.
      - Run targeted mutation tests while a layer sits above another open
        layer. Run the full branch mutation gate once it reaches the bottom.

      "Useful on its own" connects back to the
      [behaviour contract](/techniques/instructing-agents/behaviour-contracts/):
      each layer is a vertical slice with a production caller, never "add
      the types" followed by "use the types".

      ## Long-running and scheduled work

      Not everything belongs on the merge path:

      | Workflow | When | Why not a merge gate |
      | --- | --- | --- |
      | Payment sandbox e2e | Nightly, 03:17 UTC | Third-party sandboxes are slow and sometimes down |
      | Email sandbox e2e | Nightly, 03:47 UTC | Same, for email providers |
      | Production suite | Nightly, 04:23 UTC | Backup drills, races and bundle smoke tests are slow |
      | Mutation testing | Manual, per module | Far too slow for every push |
      | Specification evidence | Manual | Screenshots for each story, keyed by its stable ids |
      | Biome fix | Manual | Opens a pull request with lint fixes for a branch |

      ## Finish by rewriting the description

      > Make the finished PR name and describe what was actually built, in
      > plain words.

      Agents write the description at the start, from the plan, and never
      update it. By merge time it describes a different change. The final
      description must name the behaviour added or replaced, the
      production caller, the old path deleted, the line counts, the tests
      and mutation commands run, and any change to the pure list or the
      declared machines (the [`contract:diff`](https://github.com/chobbledotcom/tickets/blob/main/scripts/contract-drift.ts) output goes here).
---
