---
name: Approach
title: The approach
meta_title: The approach | Chobble Coding Standards
meta_description: >-
  Why the checks exist: LLM agents are fast and literal, so every rule that
  matters needs a written reason, a mechanical check, and a test that proves
  the check works.
permalink: /approach/
eleventyNavigation:
  key: Approach
  order: 2
blocks:
  - type: markdown
    content: |
      # The approach

      I build software with LLM coding agents doing most of the typing. That
      changes what a coding standard is for. A human team needs standards so
      people can read each other's code. An agent team needs standards
      because an agent will do *exactly what makes the current task look
      finished*, and nothing else.

      ## What agents get wrong, reliably

      These are not occasional slips. Every one of them happened often
      enough in my repositories that it now has a rule and a check.

      - **Copying instead of sharing.** It is quicker to paste a function and
        change one value than to find the existing one and add a parameter.
      - **Swallowing errors.** A failing test goes green fastest when the
        error is caught and turned into `null`, `[]` or a default.
      - **Weak tests.** `expect(result).toBeDefined()` passes for almost any
        bug. So does a test that sets a value and reads it back.
      - **Widening the exceptions.** A check fails, and the agent adds the
        file to the ignore list. The check still runs and now proves less.
      - **Keeping dead code.** An export "might be useful later", or exists
        only because a test imports it.
      - **Building the easy half.** The types and helpers land, and the
        hard behaviour is left for "a later pull request".
      - **Fixing the wrong thing.** A flaky runner or a broken tool gets a
        retry loop or a workaround, and the real problem is buried.
      - **Narrating.** Comments that repeat the code, commit messages that
        tell a story, and documentation that hedges every sentence.

      ## The four layers

      Each rule that matters gets as many of these layers as it can carry.

      1. **A written rule with its reason.** It lives in AGENTS.md or a
         document AGENTS.md links to. The reason matters, because the
         preferences are "defaults, not laws", and an agent can only bend a
         rule sensibly if it knows what the rule is for.
      2. **A mechanical check.** If a machine can judge the rule, a script
         judges it in precommit and in CI. Agents do not argue with a red
         check the way they argue with a sentence.
      3. **A ratchet.** Most checks start on a codebase that already breaks
         them. The existing breaks go into a baseline that can only shrink,
         so the check is strict for new code from day one.
      4. **A test of the test.** Coverage proves the tests run every line.
         Mutation testing proves they would notice if a line changed. Model
         checking proves the state machines are safe for every run, not
         only the runs someone thought to write down.

      ## Principles that run through everything

      - **Fail loudly.** Never default away a missing value. A crash with a
        clear message is cheaper than a wrong answer that looks right.
      - **One source of truth.** Declare a thing once, as data, and derive
        everything else from it. Two copies of a fact will drift.
      - **Lists that only shrink.** Every exception list, baseline and
        registry is a ratchet. Adding to one is an owner decision, never a
        side effect of making a check pass.
      - **The code is the authority once it exists.** A plan governs work
        that is not built yet. After that, check every claim against the
        real source and tests.
      - **Hardest first.** Build the part most likely to change the design
        first, so the easy parts are built on the right shape.

  - type: split-callout
    subtitle: The trap to avoid
    content: |
      ## Checks are not a substitute for design

      Mutation testing can show that an assertion is weak. It cannot show
      that a state, a race or a provider failure was never designed. That is
      why the [behaviour contract](/techniques/instructing-agents/behaviour-contracts/)
      comes before any code, and why a human approves it.

      The checks keep a good design good. They do not make a bad design
      good.
    figure_icon: "mdi:alert-circle-outline"
    figure_name: Design first, then check
    figure_subtitle: Plan the states, failures, retries and races. Then let the machines hold the line.
    figure_variant: secondary

  - type: markdown
    content: |
      ## Where to start if you want to copy this

      You do not need all of it. In my experience the order of value for
      effort is roughly:

      1. **A single AGENTS.md** with pre-decided answers. Costs an afternoon.
      2. **A strict linter config**, run read-only in CI.
      3. **Copy-paste detection at 0%**, with a low token threshold.
      4. **Ratchets** on anything that starts dirty.
      5. **100% coverage**, with an exclusion list that only shrinks.
      6. **Weak-assertion checks** on test files.
      7. **Mutation testing** on the files a branch changes.
      8. **Acceptance specs** in plain language for the user journeys.
      9. **Nightly runs against real sandboxes** for anything that talks to
         a third party.
      10. **State machines and model checking** for anything that moves
          money or cannot be allowed to lose data.

      The last few cost real effort. They earn it in code where a bug costs
      someone money.

  - type: link-button
    text: Browse all techniques
    href: /techniques/
    variant: primary
    size: lg
---
