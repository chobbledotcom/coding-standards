---
name: Cucumber acceptance specs
subtitle: User stories in plain words, with exact assertions underneath
guide-category: testing
order: 4
meta_description: >-
  How the tickets repository writes Cucumber acceptance specifications:
  three test categories, tagged Features, Rules and Scenarios, real
  rendered forms, typed tables, and a checklist for migrating old e2e
  tests without losing claims.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Why stories at all

      Unit tests prove functions. Nobody but a developer can read them, and
      an agent can write thousands without anyone noticing that the
      product's actual rules were never written down. Cucumber Features fix
      that: each one is a user story in plain English that a non-developer
      can read and argue with, backed by step code that makes exact,
      mutation-resistant assertions.

      The tickets repository has nearly a hundred Features in
      [`specs/`](https://github.com/chobbledotcom/tickets/tree/main/specs),
      and the rules for them are in
      [E2E_TESTS.md](https://github.com/chobbledotcom/tickets/blob/main/E2E_TESTS.md).

      ## Three kinds of test, and no generic e2e bucket

      > Every test falls into exactly one of three categories.

      1. **Pure unit and property tests.** Data in, data out. No database,
         network, filesystem, DOM, subprocess or real clock.
      2. **Direct technical contract tests.** The smallest real boundary:
         SQLite, HTTP, a provider transport, WebCrypto, the build output.
         These own SQL constraints, migrations, transactions, protocols,
         query budgets and concurrency.
      3. **Cucumber acceptance specs.** One user story or business rule per
         Feature, in domain language only. "No SQL, route names, field
         names, selectors, mocks, or provider payloads."

      A Cucumber story is never the only coverage of a production line.
      Direct Deno tests keep coverage at 100% on their own.

  - type: split-code
    subtitle: A real Feature
    content: |
      ## What one looks like

      This story describes a real failure mode: SumUp took the money, the
      callback never arrived, and the customer left. It reads as a business
      rule. The step code behind "the payment is still booked exactly once"
      checks the exact stored rows.

      Every Feature carries a unique `@story:` id, an owner, a risk level,
      actors and editions. Every Rule and Scenario has its own stable id.
    figure_filename: specs/payments/a-payment-with-no-callback.feature
    figure_language: gherkin
    figure_code: |
      @story:payments.a-payment-with-no-callback
      @owner:payments @risk:high
      @actor:customer @actor:organiser
      @edition:managed @edition:self-hosted
      Feature: A payment whose callback was lost

        @rule:payments.a-lost-callback-is-recovered-by-scheduled-maintenance
        Rule: Maintenance books a paid checkout whose callback never arrived

          @case:payments.maintenance-books-the-lost-payment
          Scenario: The callback never arrives and the customer never comes back
            Given Alice pays for a 45.00 Concert place through SumUp on the public booking page
            And the customer did pay SumUp
            And the site never hears that the payment arrived
            When scheduled maintenance runs
            Then the payment is booked
            And the Concert has earned 45.00
            When the customer opens the payment link they were given
            Then they reach the page that says their payment was successful
            And the payment is still booked exactly once

  - type: markdown
    content: |
      ## Writing rules that keep stories honest

      - **Plain words, exact checks.** "Keep exact mutation-resistant
        assertions in the TypeScript step definition. A plain-language
        `Then the payment is refunded once` can assert the exact provider
        call count, stored note, terminal result, and lack of a duplicate."
      - **Drive through the real rendered form.** A step that submits an
        admin edit reads the production HTML, parses its fields and CSRF
        token, and posts exactly what a browser would. Building the request
        from database rows would skip the very layer the story exists to
        prove.
      - **Validate tables at the boundary.** Every DataTable cell starts as
        a string. Parse it with a typed schema. "Never cast and hope."
      - **Business setup in Given steps, not hooks.** Hooks hold only
        technical fixture work a reader does not need.
      - **No ambiguity, no retries.** Undefined, ambiguous, pending and
        skipped steps fail. Unused step definitions fail the full run. "Do
        not add retries; a pass after retry is still a flaky failure."
      - **No speculative vocabulary.** A new shared step must be used by the
        current story or one immediately included with it.
      - **Use the site's words.** A story calls a thing what its label calls
        it, copied from the message catalog.

      ## Strict tags, checked

      `deno task specs:check` parses every Feature and validates the tags
      against a registry: exactly one globally unique `@story:` per
      Feature, one `@rule:` per Rule, one `@case:` per Scenario (or a
      `case_id` column for outlines), and known owner, risk, actor and
      edition values. Ad-hoc tags are forbidden. Those ids are durable: the
      repository's screenshot evidence is keyed by them, and they never move
      when a scenario is renamed.

      ## Migrating an old test without losing claims

      The most valuable section of E2E_TESTS.md is a checklist that exists
      because the same mistake kept happening:

      > A migration is a **replacement**, and the risk is always the same:
      > the story reads better than the test it replaced while quietly
      > proving less.

      The rule is to build the list of claims from the *old* file, as it
      stood on the merge base, never from the new story:

      ```bash
      git fetch origin main
      base=$(git merge-base origin/main HEAD)
      git diff --name-only --no-renames --diff-filter=a "$base" HEAD
      file=test/features/admin/settings-email.test.ts
      git diff "$base" HEAD -- "$file"
      git show "$base:$file"
      git diff "$base" HEAD -- "$file" | grep '^-' | grep -E 'expect|assert'
      ```

      Each claim then lands in exactly one place: in the story, in a direct
      test, or deliberately dropped and said out loud. The document lists
      claims that were lost in real migrations, including "the double-entry
      conservation check after each money action" and "that a race was
      settled by capacity rather than by one request simply failing". One
      batch lost three more claims *after* the checklist existed, because
      the author built the list from the new files.

      That last detail is typical of working with agents. A finished story
      reads as if it covers everything. Only the old file can say what is
      missing, so the procedure forces the agent to read it.
---
