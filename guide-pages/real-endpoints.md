---
name: Nightly runs against real endpoints
subtitle: Real browsers, real payment sandboxes, real email APIs, every night
guide-category: testing
order: 5
meta_description: >-
  How the tickets repository runs nightly end-to-end tests against the real
  Stripe, Square and SumUp sandboxes and real email provider APIs, plus a
  nightly production-shape suite, without making any of it a pull request
  gate.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A [mock and a sandbox](/glossary/#mock) are different things: a mock is a fake
      service, a sandbox is a real provider account that moves no real money.
      The providers here have sandbox docs:
      [Stripe](https://docs.stripe.com/testing),
      [Square](https://developer.squareup.com/docs/devtools/sandbox/overview) and
      [SumUp](https://developer.sumup.com/). The browser is driven by
      [Playwright](https://playwright.dev/docs/intro), and the public URL comes
      from a [Cloudflare quick tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/do-more-with-tunnels/trycloudflare/).
  - type: markdown
    content: |
      ## What mocks cannot catch

      The main test suite talks to [stripe-mock](https://github.com/stripe/stripe-mock)
      and to stubbed Square, SumUp and email responses. That makes it fast
      and deterministic, and it runs on every pull request. It also means
      the suite can only ever agree with what *we think* the providers do.

      The one class of bug mocks cannot find is drift between our code and
      the real provider: an API field that changed, a redirect URL the
      provider rejects, a webhook that arrives in a different shape, a
      sandbox card page that moved its button. For anything that takes
      money or sends mail, that class of bug matters most.

      ## The nightly payment run

      [`e2e-payments/`](https://github.com/chobbledotcom/tickets/tree/main/e2e-payments)
      is a separate harness. For each provider it:

      1. boots the **real** app server ([`src/index.ts`](https://github.com/chobbledotcom/tickets/blob/main/src/index.ts)) on a fresh
         file-backed database;
      2. exposes it to the internet through a
         [cloudflared](https://github.com/cloudflare/cloudflared) quick
         tunnel, so the provider can reach its webhooks and return URLs;
      3. drives a **real Chromium** through a full paid booking, typing a
         sandbox test card into the provider's own hosted checkout page;
      4. checks the admin side shows the booking as paid, with the right
         income per listing;
      5. issues a real sandbox refund, and checks it lands once.

      It is orchestrated by the same [Cucumber](https://cucumber.io/docs/) runner as the main specs.
      [`live-payment-providers.feature`](https://github.com/chobbledotcom/tickets/blob/main/e2e-payments/specs/live-payment-providers.feature)
      is the human contract. Its scenarios include a refund where the
      provider succeeds while the local database write fails, a replayed
      SumUp callback with forged ids that must be refused without a read, a
      price change while a visitor is mid-checkout, and a complex order of a
      package, a member and a plain listing paid in one go.

      Every scenario gets its own identity (booker email, listing names,
      owner credentials, never the defaults), so a leftover from one run can
      never satisfy an assertion in the next.

  - type: split-code
    subtitle: The schedule
    content: |
      ## Nightly, and on demand

      The workflow runs at 03:17 UTC every night, and can be started by
      hand. Off the hour, so it does not queue behind every other scheduled
      job on GitHub.

      Each provider is its own leg, with `fail-fast: false`, so one broken
      sandbox does not hide the others.
    figure_filename: .github/workflows/payment-sandbox-e2e.yml
    figure_language: yaml
    figure_code: |
      # It is deliberately NOT a PR gate: it needs
      # network to third-party sandboxes and is
      # inherently slower/flakier than mocked tests.
      # It runs nightly and on demand. Every leg must
      # execute: a missing provider secret fails its
      # leg.

      on:
        workflow_dispatch:
        schedule:
          # 03:17 UTC nightly (off-peak, avoids the
          # top-of-hour rush).
          - cron: "17 3 * * *"

  - type: markdown
    content: |
      ## Driving the browser like a person

      The harness fills fields by their accessible name and clicks buttons
      by their visible text, the way a person would. It never forces a
      click and never calls `form.submit()`. Some headless Chromium builds
      stall after a form POST. The harness detects that and falls back to
      `requestSubmit`, which still runs browser validation and the app's own
      handlers, and it only does so when a page-side witness proves the
      first click never fired. A refund form can never be submitted twice
      by the fallback.

      ## Nightly email, with different secret rules

      A second workflow, [`email-sandbox-e2e.yml`](https://github.com/chobbledotcom/tickets/blob/main/.github/workflows/email-sandbox-e2e.yml), runs at 03:47 UTC. It
      sends real requests through the production email code to [Mailgun](https://documentation.mailgun.com/) (EU
      and US), [Postmark](https://postmarkapp.com/developer), [Resend](https://resend.com/docs) and [SendGrid](https://www.twilio.com/docs/sendgrid): one single send with an
      attachment and reply-to, and one bulk send with unsubscribe
      substitution. Every message goes to a safe address, such as a
      provider's discard sink.

      The two runs treat a missing secret differently, on purpose:

      - **Payments:** a missing provider secret *fails* its leg. The
        contract is that every provider is tested every night.
      - **Email:** a missing on-switch secret *skips* that leg and reports
        it. Operators enable exactly the providers they hold credentials
        for. A leg whose secret is present must pass.

      ## The nightly production-shape suite

      [`deno task suite:production`](https://github.com/chobbledotcom/tickets/blob/main/scripts/production-suite/run.ts) runs at 04:23 UTC. It gathers the checks
      that ask production-shaped questions and are too slow or too noisy to
      gate every pull request:

      - an edge-bundle boot smoke test of the real deployment artefact;
      - backup and restore drills;
      - a concurrent-reservation race;
      - webhook replay and idempotency;
      - a `deno audit --level=critical` security report.

      ## Catching tomorrow's failure today

      A schedule-only run has a weakness: a change that breaks it merges
      green and fails the next night. The cheapest of those to catch is a
      renamed button. So [`deno task check:e2e-labels`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-e2e-labels.ts) runs in every
      precommit and checks that every label the payment harness clicks or
      asserts is real copy from the message catalog. A copy rename now
      fails the pull request that made it, not the next night's run.

  - type: callout
    variant: info
    icon: "mdi:lightbulb-on-outline"
    name: Keep real-world tests out of the merge path
    content: |
      Tests against third parties are slow and occasionally flaky for
      reasons that are not yours. Gating merges on them teaches everyone,
      agents included, to re-run until green. Run them on a schedule,
      report failures loudly (a harness crash goes to [Sentry](https://docs.sentry.io/)), and move
      every cheap, deterministic part of them into the main gates.
---
