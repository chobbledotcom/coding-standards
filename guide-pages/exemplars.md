---
name: Exemplars over essays
subtitle: Point agents at the best code you have, and tell them to copy its shape
guide-category: instructing-agents
order: 5
meta_description: >-
  How docs/designing-systems.md steers new features by naming exemplar
  files for each quality: schema-first, pure core, curried helpers, cold
  start budgets, efficient SQL and late decryption.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      An **exemplar** is a file in the codebase that shows a pattern done well.
      Terms on this page: [schema validation](/glossary/#schema) with
      [Valibot](https://valibot.dev/guides/introduction/),
      [pure functions](/glossary/#pure-function), [currying](/glossary/#currying) and
      [cold starts](/glossary/#bunny). The [Valibot guide](https://valibot.dev/guides/introduction/)
      is short and a good first read.
  - type: markdown
    content: |
      ## Agents copy what they see

      An agent asked to build a feature looks for something similar and
      copies it. That is good when it finds the best example in the
      codebase and bad when it finds the oldest. So the design guide does
      not describe good design in the abstract. It names files.

      [docs/designing-systems.md](https://github.com/chobbledotcom/tickets/blob/main/docs/designing-systems.md)
      opens:

      > When planning a new feature, design it to have every quality below.
      > Each one names reference implementations in this codebase — read the
      > exemplar before designing, and copy its shape rather than inventing a
      > new one. These are the systems we want more of.

      ## The qualities, and their exemplars

      - **Schema-tized.** A [valibot](https://valibot.dev/guides/introduction/) schema is the single source of truth
        for a value type, and the [TypeScript](https://www.typescriptlang.org/docs/) type, the runtime guard and the
        options list all derive from it. Tables come from `defineTable`,
        admin APIs from [`defineCrudApi`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/rest/crud-api.ts#L74), forms from [`defineForm`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/forms/definition.ts#L89). "If your
        plan contains a hand-rolled dispatcher, an ad-hoc form, bespoke CRUD
        routes, or hand-written row (de)serialization, stop: there is a
        `define*` factory for that already."
      - **Checked forwards and backwards.** Lifecycles are
        [machines](/techniques/machines-and-proofs/state-machines/), checked
        by [models](/techniques/machines-and-proofs/tla-and-apalache/), with
        stored data checked separately by a live scan.
      - **Pure, functional.** The core is data in, data out, and IO is a thin
        shell around it. The exemplar is
        [`largest-remainder.ts`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/largest-remainder.ts):
        "a complete allocation algorithm with zero imports; the hardest
        logic in the money paths and the easiest to test".
      - **Modularised.** One concept per file, one layer per directory.
      - **Well-named.** "If you cannot name the file in a couple of words,
        it is probably two concepts — split it."
      - **Built for cold starts.** Nothing heavy at module load, lazy
        singletons, request-scoped caches, and a hard budget of 50
        subrequests per request on Bunny's edge.
      - **Efficient SQL.** Invariants enforced inside the mutating statement,
        trigger-maintained aggregates, `UPDATE … RETURNING`.
      - **Decrypt only what you need.** Blind HMAC index columns for
        lookups, one encrypted PII blob per attendee, decrypted only at
        render time.

  - type: split-code
    subtitle: One declaration, everything derived
    content: |
      ## The schema pattern

      This is the shape the guide asks for every closed set of values. The
      type, the guard and the list of options cannot disagree, because
      there is only one of them.
    figure_filename: docs/designing-systems.md
    figure_language: typescript
    figure_code: |
      export const ContactFieldSchema = v.picklist([
        "email",
        "phone",
        "address",
        "special_instructions",
      ]);
      export type ContactField =
        v.InferOutput<typeof ContactFieldSchema>;
      export const CONTACT_FIELDS = ContactFieldSchema.options;
      export const isContactField = (s: string): s is ContactField =>
        v.is(ContactFieldSchema, s);

  - type: markdown
    content: |
      ## Plain language for functional code

      The codebase is functional and curried, built on a small
      [`#fp`](https://github.com/chobbledotcom/tickets/blob/main/src/fp.ts)
      module. A separate rule stops that turning into jargon:

      > Keep the functional style, but name helpers and write comments in
      > simple domain words, not CS jargon. A helper must explain itself like
      > "Keeps only children that can still be booked for this ticket."

      ## Do not reinvent the wheel

      The guide also lists what to use instead of writing it: valibot for
      validation, `@std/collections` for collections, `Temporal` for dates
      and time zones, `Intl` for formatting. When the house calling
      convention differs from a library's, write a thin adapter, and
      document the reason when hand-rolling is unavoidable.

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      For each quality you care about, find the one file in your codebase
      that shows it best, and write its path next to the quality. Update the
      list when a better example lands. An agent will copy the exemplar far
      more faithfully than it will follow a paragraph of advice.
---
