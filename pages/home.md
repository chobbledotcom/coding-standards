---
name: Home
title: Chobble Coding Standards
permalink: "/"
meta_title: Chobble Coding Standards | Keeping code and LLMs honest
meta_description: >-
  A field guide to the rules, checks, tests and proofs I use to build
  software with LLM agents: AGENTS.md, ratchets, jscpd, knip, 100% coverage,
  mutation testing, Cucumber specs, nightly sandbox runs and Apalache models.
eleventyNavigation:
  key: Home
  order: 1
blocks:
  - type: hero
    badge: A field guide
    content: |
      # Keeping code, and the LLMs that write it, on the straight and narrow

      Most of the code in my projects is now written by LLM agents. Agents
      are fast, tireless and keen, and left alone they will copy a function
      instead of sharing it, swallow an error to make a test pass, and quietly
      add one more entry to the list of things the checks ignore.

      This site explains every technique I use to stop that: the written
      rules, the machines that enforce them, and the tests and proofs that
      catch what slips past both.
    buttons:
      - text: Browse the techniques
        href: /techniques/
        variant: primary
        size: lg
      - text: Start with the philosophy
        href: /approach/
        variant: secondary
        size: lg

  - type: stats
    items:
      - "100%|Line and branch coverage"
      - "0%|Tolerated duplication"
      - "100%|Mutation kill rate on changed code"
      - "25|Precommit gates"

  - type: markdown
    content: |
      ## The short version

      Every rule I care about lives in one place,
      [AGENTS.md](https://github.com/chobbledotcom/tickets/blob/main/AGENTS.md).
      Every rule that a machine can check has a checker. Every checker that
      starts with existing debt holds that debt in a list that is only
      allowed to shrink. The tests have to cover every line, and then a
      mutation tester proves that the tests would notice if a line changed.
      The parts that move money are drawn as state machines and checked by a
      model checker.

      None of this is exotic. It is a pile of small, boring, mechanical
      checks, each of which closes one door that an eager agent likes to
      walk through.

  - type: features
    intro_content: |
      ## What the guide covers
      Each area has its own section, with worked examples from real code.
    items:
      - icon: "mdi:book-open-page-variant-outline"
        name: Instructing the agents
        description: >-
          One rulebook, a planning workflow that comes before any code, and
          guardrails that stop an agent "fixing" the things it should report.
      - icon: "mdi:shield-check-outline"
        name: Static checks and ratchets
        description: >-
          Linting, copy-paste detection, dead-code hunts, purity checks and
          baselines that can only ever get smaller.
      - icon: "mdi:test-tube"
        name: Testing that bites
        description: >-
          Full coverage, weak-assertion gates, mutation testing, Cucumber
          stories and nightly runs against real payment and email sandboxes.
      - icon: "mdi:graph-outline"
        name: Machines and proofs
        description: >-
          Lifecycles declared as data, swept against the real code, and
          checked for every possible run by the Apalache model checker.
      - icon: "mdi:source-pull"
        name: Workflow and gates
        description: >-
          The precommit pipeline, CI that mirrors it, stacked pull requests,
          review rules and an LLM grader that knows the house style.
      - icon: "mdi:pin-outline"
        name: A pinned environment
        description: >-
          Nix and devenv pin every tool, so a check that passes on my machine
          passes in CI for the same reason.

  - type: guide-categories

  - type: split-code
    subtitle: What a rule looks like
    content: |
      ## Rules that pre-decide the argument

      The rules in AGENTS.md are written to end discussions before they
      start. An agent that has to ask "should I delete this unused export?"
      or "should I merge these two near-identical helpers?" will often ask,
      or worse, guess. The rulebook answers in the heading.

      Each preference is one bold line and a short reason, so an agent can
      apply it and a person can argue with it.
    button:
      text: Read about AGENTS.md
      href: /techniques/instructing-agents/agents-md/
    figure_filename: AGENTS.md
    figure_language: markdown
    figure_code: |
      - **Remove dead code — always the answer**: When code
        has no production caller, delete it. Never keep it
        "for symmetry" or "for future use".
      - **Unify systems — the answer is yes**: When two
        systems do almost the same job, find the single
        abstraction that does both.
      - **Never loosen a check to close a finding**: Fix
        the code, or argue on the thread that the finding
        is wrong.

  - type: markdown
    content: |
      ## Where the examples come from

      Almost every example on this site comes from
      [chobbledotcom/tickets](https://github.com/chobbledotcom/tickets), an
      open-source ticket booking system that runs on [Bunny Edge Scripting](https://bunny.net/edge-scripting/) and
      takes real payments through [Stripe](https://docs.stripe.com/testing), Square and [SumUp](https://developer.sumup.com/). A few come from
      [chobbledotcom/chobble-template](https://github.com/chobbledotcom/chobble-template),
      the [Eleventy](https://www.11ty.dev/docs/) template this site is built on. Both repositories are
      public, so every file named here is a link you can follow and read in
      full.

  - type: callout
    variant: info
    icon: "mdi:account-hard-hat-outline"
    name: Available for hire
    content: |
      I'm Stefan, and I build software this way for clients. If you want
      these checks in your own codebase, or a developer who works like
      this, see [software development at Chobble](https://www.chobble.com/services/software-developer/)
      or [get in touch](https://www.chobble.com/contact/).

  - type: cta
    content: |
      ## New to all this?

      Start with the philosophy page. It explains why each family of checks
      exists, and which ones give the most value for the least effort if you
      want to adopt only a few. Every technique page opens with a short
      "New to this?" note, and the [glossary](/glossary/) explains each tool
      and idea in plain words, with links to its documentation.
    button:
      text: Read the approach
      href: /approach/
---
