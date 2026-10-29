---
name: Techniques
title: Techniques
subtitle: Every technique, grouped by what it protects
meta_title: Techniques | Chobble Coding Standards
meta_description: >-
  The full list of techniques: agent instructions, static checks and
  ratchets, testing, state machines and proofs, and the workflow that ties
  them together.
permalink: /techniques/
eleventyNavigation:
  key: Techniques
  order: 3
blocks:
  - type: markdown
    content: |
      # Techniques

      The techniques fall into five groups. They overlap on purpose: a rule
      in AGENTS.md is usually backed by a static check, the static check is
      backed by a test, and the test is backed by mutation testing. Each
      layer catches what the one above lets through.

  - type: guide-categories

  - type: callout
    variant: info
    icon: "mdi:lightbulb-on-outline"
    name: If you only adopt three things
    content: |
      1. **One rulebook** that every agent reads, with the answer to each
         common argument written into the rule. See
         [AGENTS.md](/techniques/instructing-agents/agents-md/).
      2. **Ratchets** on every check that starts with debt, so the debt can
         only go down. See [Ratchets](/techniques/static-checks/ratchets/).
      3. **Mutation testing** on changed code, so a test suite with 100%
         coverage cannot be 100% hollow. See
         [Mutation testing](/techniques/testing/mutation-testing/).
---
