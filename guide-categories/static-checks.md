---
name: Static checks and ratchets
subtitle: Lint, duplication, dead code, purity and lists that only shrink
order: 2
icon: "mdi:shield-check-outline"
meta_description: >-
  The static checks that run before every commit: Biome, jscpd and its
  variants, knip and dead-code scans, offensive-programming checks, effect
  and import checks, comment limits and the ratchets that hold them.
blocks:
  - type: guide-header
  - type: markdown
    content: |
      Static checks are the cheapest layer. They run in seconds, they never
      flake, and an agent cannot talk its way past them. The trick is to
      make each one strict for new code on the first day, even when the
      existing code breaks it. That is what the ratchets are for.
  - type: guide-pages-list
---
