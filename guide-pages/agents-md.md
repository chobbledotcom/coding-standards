---
name: AGENTS.md
subtitle: One rulebook that every agent reads, with the answers already written in
guide-category: instructing-agents
order: 1
meta_description: >-
  How the tickets repository uses a single AGENTS.md as the one source of
  guidance for every coding agent, with preferences that pre-decide the
  common arguments.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      Coding agents such as [Claude Code](https://www.anthropic.com/claude-code)
      read instruction files from your repository before they start work.
      [agents.md](https://agents.md/) describes the shared `AGENTS.md`
      convention. You can read the [whole tickets AGENTS.md](https://github.com/chobbledotcom/tickets/blob/main/AGENTS.md)
      in about fifteen minutes, and it is worth doing before you read the rest
      of this site. See also: [LLM coding agent](/glossary/#llm-agent).
  - type: markdown
    content: |
      ## One file, linked outwards

      Every coding tool has its own idea of where instructions live:
      [`CLAUDE.md`](https://github.com/chobbledotcom/tickets/blob/main/CLAUDE.md), `AGENTS.md`, `.cursorrules`, a skills folder. In my
      repositories there is one file,
      [AGENTS.md](https://github.com/chobbledotcom/tickets/blob/main/AGENTS.md),
      and the others point at it. The whole of the tickets `CLAUDE.md` is
      one line:

      ```markdown
      @AGENTS.md
      ```

      AGENTS.md is the root of a small tree. It links to longer documents
      for the topics that need room: the
      [pull request workflow](https://github.com/chobbledotcom/tickets/blob/main/PR_WORKFLOW.md),
      the [acceptance test rules](https://github.com/chobbledotcom/tickets/blob/main/E2E_TESTS.md),
      [designing new systems](https://github.com/chobbledotcom/tickets/blob/main/docs/designing-systems.md)
      and [mutation testing](https://github.com/chobbledotcom/tickets/blob/main/docs/mutation-testing.md).
      Nothing an agent needs is anywhere else, and the file says so:

      > All repository guidance is in this file or in documents that it
      > links to. Read this file and follow its links. Do not search the
      > wider machine for instructions, plans, or approval records. If
      > required guidance is absent, ask the user for its location.

      That paragraph exists because agents wander. Given a vague task, an
      agent will go looking for context in sibling folders, old plan files
      and its own memory, and treat whatever it finds as instructions. A
      closed world of guidance makes the agent's behaviour predictable, and
      it makes the rules reviewable in one pull request.

      ## Preferences are defaults, not laws

      The core of the file is a list of preferences. It opens like this:

      > These preferences are defaults, not laws. When the letter of a rule
      > and its purpose pull apart, serve the purpose. Say which rule you
      > bent and why.

      LLMs follow rules to the letter. A rule like "keep files under 400
      lines" will, sooner or later, produce a file split in half at line
      399 with no thought for what belongs together. Giving each rule its
      purpose lets the agent apply it sensibly. Asking it to *say which rule
      it bent* turns a silent judgement call into something a reviewer can
      see and challenge.

      ## Pre-decide the argument

      Many preferences carry their verdict in the heading. These are real
      headings from the file:

      - **Remove dead code — always the answer**
      - **Unify systems — the answer is yes**
      - **Hardest first, no need to ask**
      - **Always the complete version**
      - **Never loosen a check to close a finding**
      - **One path for one-or-many**
      - **A written-down diagnosis is a hypothesis, not a finding**

      Each one answers a question an agent would otherwise stop to ask, or
      answer badly on its own. "Should I keep this helper in case it is
      useful later?" has an answer. "Should I build the single-item version
      first and the list version later?" has an answer. An agent that does
      not have to ask moves faster and drifts less.

      ## Short rules, with the reason attached

      Each preference is one bold phrase and two or three sentences. Here
      is a typical one:

      > **Do not defend against the impossible**: No fallbacks or catches
      > for failures that need the whole app to be down. Let them throw.
      > Reserve resilience for failures that occur in normal operation.

      The pattern is: the rule, the concrete thing it forbids, and the
      exception. Long explanations go in a linked document, so the main
      file stays short enough to load into every session.

      ## Rules that name their checker

      Wherever a machine can judge a rule, the rule names the task that
      enforces it. That tells the agent how to check its own work before it
      commits, and it tells a reviewer the rule is not optional.

      | Rule in AGENTS.md | What enforces it |
      | --- | --- |
      | Zero code duplication | [`deno task cpd`](https://github.com/chobbledotcom/tickets/blob/main/scripts/cpd.ts) |
      | Keep files under ~400 lines | [`deno task check:file-lengths`](https://github.com/chobbledotcom/tickets/blob/main/scripts/counts-check.ts) |
      | Never add a coverage exclusion | [`deno task check:coverage-exclusions`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-coverage-exclusions.ts) |
      | No alias exports | [`deno task check:alias-exports`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-alias-exports.ts) |
      | Imports name a module one way | [`deno task check:imports`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-imports.ts) |
      | Comments are short | [`deno task check:comments`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-comments.ts) |
      | Simplified Technical English | [`deno task check:ste`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-ste.ts), [`check:comment-ste`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-comments-ste.ts) |
      | Simple language for users | [`deno task check:copy`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-copy.ts) |
      | No empty catch | [`deno task check:empty-catch`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-empty-catch.ts) |
      | `toBeDefined()` is blocked | [`deno task check:presence-assertions`](https://github.com/chobbledotcom/tickets/blob/main/scripts/counts-check.ts) |

      A rule with no checker is a rule that depends on the agent's mood.
      Over time, the rules that keep being broken get a checker.

      ## The file is held to its own standards

      AGENTS.md is written in Simplified Technical English and checked by
      `deno task check:ste`, like every other Markdown file in the
      repository. That keeps it short and unambiguous, and it means the
      agents that edit it are held to the same writing rules they apply
      everywhere else. See [Plain words](/techniques/instructing-agents/plain-words/).

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      Start with one AGENTS.md. Point every other tool's instruction file
      at it. Write each rule as a bold heading that contains the answer,
      then two sentences of reason. Every time an agent makes the same
      mistake twice, add a rule; every time it breaks a rule twice, add a
      check.
---
