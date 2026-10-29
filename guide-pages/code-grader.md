---
name: An LLM code grader
subtitle: The repository's own checks plus an LLM for the judgement calls
guide-category: workflow
order: 3
meta_description: >-
  How deno task grade:code scores source files against the rules in
  AGENTS.md: mechanical checks reused from precommit, judgement questions
  asked of an LLM with line-addressed evidence, and low-confidence answers
  turned into warnings.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## The half the scripts cannot do

      Many rules in AGENTS.md have a mechanical half and a judgement half.
      A script can count the lines in a comment. It cannot tell whether the
      comment says anything. A script can list every `??` in a file. It
      cannot tell which ones paper over a missing value and which guard a
      genuinely optional one.

      `deno task grade:code` covers both halves for one file, a folder, or
      every module under `src/`, and prints a scored report. It is advice,
      not a gate.

      ## Mechanical checks, reused

      The mechanical half does not re-implement the rules. It imports the
      repository's own checkers
      ([`scripts/grade-code/checks.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/grade-code/checks.ts)):

      ```typescript
      import { findIssues as findAliasExportIssues } from "#scripts/check-alias-exports/rules.ts";
      import { findCommentIssues } from "#scripts/check-comments/rules.ts";
      import { findIssues as findEmptyCatchIssues } from "#scripts/check-empty-catch/rules.ts";
      import { LINE_LIMIT } from "#scripts/check-file-lengths/rules.ts";
      import { findImportIssues } from "#scripts/check-imports/rules.ts";
      ```

      So a mechanical grade always agrees with what `deno task precommit`
      would say. That is only possible because every checker keeps its
      rules in a pure `rules.ts`, separate from the IO that reads files.

      ## Judgement questions, with evidence

      The judgement half asks an LLM a fixed set of questions, one call per
      file
      ([`questions.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/grade-code/questions.ts)).
      Each question is a 0–3 score with four written criteria, worst to
      best, and an instruction that names the AGENTS.md section it judges.
      Some of the questions:

      - Do comments earn their place, and describe current code?
      - Is the copy plain language, and does naming match the site?
      - Is the code functional and composed, or hand-rolled loops?
      - Is it schema over organic structure, with shared dispatch?
      - One path for one-or-many?
      - Offensive, not defensive? Do lookups throw instead of returning
        stand-ins? Are there guards for impossible states?
      - Does SQL select only needed columns, with the right transaction
        shape?
      - Is pure logic split from IO?

      The model is not asked to find things on its own. The mechanical
      pass extracts line-addressed evidence first: every fallback operator,
      every catch clause, every cast, every SQL string, every link. The
      question then asks the model to judge *those* lines. From the
      "offensive errors" question:

      > `fallback_operators` lists every `??`, `||`, and `?.` with its line;
      > `catch_clauses` lists catches. Score each down when it papers over a
      > value the code treats as expected [...] and score `??`/`||`/`?.` on
      > genuinely optional values as fine.

      Grounding the model in extracted evidence keeps its answers about the
      file in front of it, rather than about code in general.

      ## Uncertainty becomes a warning

      Every answer comes back with a confidence. A pass or a fail that the
      model itself is unsure about (confidence under 0.3) becomes a `WARN`
      for a human to read, not a verdict. A grader that is confidently wrong
      is worse than no grader.

      ## How it gets used

      - On a file before a refactor, to see which rules it breaks.
      - Across `src/` with `--csv`, to rank files worst-first and pick the
        next clean-up target.
      - With `--no-jev` for a fast, free, mechanical-only pass.
      - By an agent, on its own change, before asking for review.

  - type: callout
    variant: info
    icon: "mdi:lightbulb-on-outline"
    name: Keep rules pure so you can reuse them
    content: |
      The grader was cheap to build because each checker already kept its
      rule logic in a pure module. If your lint scripts mix rules with
      file IO, the first thing to do is split them. Then the same rule can
      run in precommit, in a grader, in an editor, and in its own unit
      tests.
---
