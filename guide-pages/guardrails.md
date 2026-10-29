---
name: Guardrails for autonomous agents
subtitle: The rules that stop an agent fixing the wrong thing
guide-category: instructing-agents
order: 3
meta_description: >-
  Rules for agents working unattended on a shared machine: report
  infrastructure problems instead of working around them, keep scratch work
  in one place, stay on task, never loosen a check, and never lose work.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      An agent working "unattended" runs commands and edits files without a
      person approving each step. That is fast, and it means mistakes pile up
      before anyone sees them. These rules are the ones that stop the most
      common pile-ups. If git terms like *staging*, *base branch* or `--no-verify`
      are new, the [Pro Git book](https://git-scm.com/book/en/v2) (free online)
      covers them in its first three chapters.
  - type: markdown
    content: |
      Several agents often work on the tickets repository at once, on one
      machine, each on its own branch. Most of the rules on this page exist
      because of something an unattended agent did with the best of
      intentions.

      ## Infrastructure problems are not yours to fix

      This is the rule that saved the most time. When an agent hits a
      broken test runner, a missing tool or a network failure, its instinct
      is to make the problem go away: a retry loop, a probe script, a tweak
      to the harness. The result is a branch full of changes nobody asked
      for, and a real infrastructure fault that nobody hears about.

      > If you think you hit an infrastructure problem, stop. Do not fix it.
      > Do not work around it: no probe scripts, no harness changes, no
      > retry loops. Leave your work as it is and wait for a human decision.

      The rule lists what counts as infrastructure (the test runner and
      harness, coverage tooling, the pre-commit hook, [devenv](https://devenv.sh/getting-started/), nix, deno, CI
      runners, the network, the machine) and what to report: the exact
      command, the exact error, and why it looks like infrastructure. When
      an `agent-report-infra` command is on the path, the agent uses it.

      It also says, plainly, where the line is: "A failure in your own
      change is still yours to fix." And it overrides two other rules that
      would otherwise pull the wrong way: do not re-run a suspected
      infrastructure failure, and do not commit broken work just to leave
      the branch tidy.

      ## Scratch work lives in `.scratch/`

      Agents like to experiment: a probe script here, a throwaway test
      there. Without a rule, those files end up in `scripts/`, in the pull
      request, or committed by accident.

      > Put every probe, experiment, throwaway test, and note in `.scratch/`
      > in your checkout. [...] You can create and delete anything in it
      > without asking. [...] Nothing in `.scratch/` is ever committed.

      Giving the agent a place where anything goes makes it much easier to
      enforce that nothing goes anywhere else.

      ## No shell bypasses

      The runtime denies `deno eval`, `bash -c`, `node -e`, `python -c`,
      throwaway `deno run -A` scripts, `kill` and `pkill`. The rule adds:
      "Never reach the same result another way." An agent that cannot run
      arbitrary code has to use the repository's own tasks, which are
      tested and scoped.

      ## Stay on your task

      > Fix only what the task names. Open an issue with the evidence when
      > you notice an unrelated bug, a coverage gap, or a flaky test. [...]
      > Never fix it in the task's pull request.

      An agent that notices a problem wants to fix it. That turns a
      ten-line pull request into a two-hundred-line one that is hard to
      review and hard to revert. An issue keeps the finding and keeps the
      pull request small. The flip side is "A branch fixes the defects it
      introduces": a review finding about code the branch added is that
      branch's job.

      ## Never loosen a check to close a finding

      > Fix the code, or argue on the thread that the finding is wrong. A
      > deliberate change to a check is an owner decision, never a side
      > effect of review pressure.

      This is the single most important rule for keeping the static checks
      meaningful. Without it, every check slowly gets weaker, one reasonable
      exception at a time. The [ratchets](/techniques/static-checks/ratchets/)
      enforce the same thing mechanically.

      ## Small habits that prevent big messes

      - **Stage what you changed, never `git add -A`.** Name the files, then
        read `git status --short` before committing. Agents leave debris.
      - **Judge nothing from a stale base ref.** Run `git fetch origin main`
        before reading an `origin/main...HEAD` diff or running the mutation
        gate. A stale base makes a clean branch look dirty, or the reverse.
      - **Never lose work.** Commit and push work in progress before a
        pause or hand-off, marked as broken if it is.
      - **A written-down diagnosis is a hypothesis.** An issue or an earlier
        agent's note says "the bug is in X". Re-derive it from the current
        source before fixing anything.
      - **Once it is built, the code is the authority.** A plan describes
        work that does not exist yet. After that, check every finding
        against `src/` and `test/`, not against the plan.
      - **No review-bot commands.** Agents do not post `@coderabbitai
        review` or similar. Every push already starts a review.

      ## Be a good neighbour on a shared machine

      The "Final check" section tells agents not to run the full precommit
      or full coverage locally. They run the tests for the files they
      changed with [`deno task test:files`](https://github.com/chobbledotcom/tickets/blob/main/scripts/run-test-files.ts), then a list of light gates
      (lint, duplication, file lengths, imports, comment language,
      typecheck). When a `job-submit` queue exists, tests go through it. The
      commit message then says
      `light gates passed; full precommit left to hosted CI`, so a reviewer
      knows exactly what ran where.
---
