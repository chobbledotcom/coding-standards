---
name: Machines and proofs
subtitle: Lifecycles declared as data and checked for every possible run
order: 4
icon: "mdi:graph-outline"
meta_description: >-
  Declaring stateful lifecycles as executable state machines, sweeping every
  cell against the real code, and proving safety with TLA+ models checked
  by Apalache.
blocks:
  - type: guide-header
  - type: markdown
    content: |
      Tests check the runs someone thought of. For code that moves money,
      that is not enough. These pages show how a lifecycle becomes a table
      the production code reads, how tests sweep every cell of that table,
      and how a model checker proves the safety rules hold for runs of any
      length.
  - type: guide-pages-list
---
