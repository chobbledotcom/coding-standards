---
name: Workflow and gates
subtitle: Precommit, CI, stacked pull requests, reviews and a pinned toolchain
order: 5
icon: "mdi:source-pull"
meta_description: >-
  The workflow that ties the checks together: the precommit pipeline,
  mirrored CI jobs, required-check manifests, stacked pull requests, review
  rules, an LLM code grader and a Nix-pinned environment.
blocks:
  - type: guide-header
  - type: markdown
    content: |
      Checks only help when they run at the right moment, on the right tree,
      with the right tools. These pages cover the order things happen in,
      from a local commit to a merged stack, and the environment that makes
      a local pass mean the same thing as a CI pass.
  - type: guide-pages-list
---
