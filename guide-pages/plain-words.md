---
name: Plain words
subtitle: Simplified Technical English for developers, simple language for users
guide-category: instructing-agents
order: 4
meta_description: >-
  How the tickets repository holds documentation, comments and commit
  messages to ASD-STE100 Simplified Technical English, and user-facing copy
  to Simple Wikipedia level, with mechanical checks for both.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      [Simplified Technical English](/glossary/#ste) is a controlled writing standard
      from the aerospace industry. "Copy" is the text a user reads in an app:
      buttons, errors, hints. A **message catalog** keeps all of that text in
      one file per language, so the code refers to it by key. See
      [what i18n means](https://developer.mozilla.org/en-US/docs/Glossary/Internationalization)
      on MDN.
  - type: markdown
    content: |
      ## Why writing rules belong in a coding standard

      LLM prose has a recognisable shape: long sentences, hedges, "-ing"
      verbs everywhere, "seamlessly leverage", and a paragraph of narration
      where one line of reason would do. In documentation it hides the
      facts. In comments it buries the one surprise worth knowing. In user
      copy it confuses the people paying for the product.

      The tickets repository uses two writing standards, and checks both.

      ## Simplified Technical English, for anything a developer reads

      Documentation, commit messages, pull request descriptions, error
      messages, log lines, script output and code comments follow
      [ASD-STE100](https://www.asd-ste100.org/), the controlled language
      written for aircraft maintenance manuals. The AGENTS.md summary:

      - Classify the passage first. Procedures use the imperative, one
        instruction per sentence, at most 20 words. Descriptions use simple
        tenses, at most 25 words.
      - Only the infinitive, imperative, simple present, simple past, simple
        future and the past participle as an adjective. No present perfect.
        No "-ing" verb forms.
      - Active voice. Only `can`, `will` and `must` as modals.
      - No contractions. No semicolons: write two sentences.
      - Keep the word "that". Put the condition before the command: "If the
        test fails, read the log."
      - One word carries one meaning through a whole document.
      - Delete words that carry no fact: `simply`, `seamlessly`,
        `leverage`, "in order to".
      - Write the command or condition of a warning first, and the risk
        second.

      STE suits agents well. It is a closed set of rules, most of them
      mechanical, and it pushes every sentence towards one fact stated
      once.

      ## Simple language, for anything a user reads

      Every message a person sees in the app (errors, hints, buttons,
      warnings, empty states) reads at roughly Simple Wikipedia level. All
      of it lives in a message catalog, `src/locales/en/*.json`, reached
      through `t("key")`, so "changing what a user reads is a catalog edit,
      not a template edit".

      The rules are short: one idea per sentence, everyday words,
      front-load the action, active voice addressed to "you", no
      implementation jargon, sentence case, and errors that state both the
      problem and the fix. Some answers are fixed so they never drift:

      > A confirm-by-typing error is always `"<Thing> name does not match.
      > Please type the exact name to confirm."`

      ## The checks

      | Task | What it reads | What it checks |
      | --- | --- | --- |
      | [`check:ste`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-ste.ts) | Every Markdown file | Mechanical STE rules, against per-document baselines that only fall |
      | [`check:comment-ste`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-comments-ste.ts) | Every comment in `src/` | Contractions, banned modals, present perfect, semicolons, sentences over 25 words |
      | [`check:copy`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-copy.ts) | The locale catalog | The mechanical simple-language rules |
      | [`check:e2e-labels`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-e2e-labels.ts) | The nightly payment e2e | Every label it clicks is real catalog copy |

      [`scripts/check-ste/rules.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-ste/rules.ts)
      and
      [`scripts/check-copy/rules.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-copy/rules.ts)
      are pure modules: copy in, issues out. The copy checker states its own
      scope honestly:

      > The rules here are the *mechanical* half of the "Simple Language"
      > guide in AGENTS.md — the parts a machine can judge without reading
      > for tone. They stay deliberately narrow (few false positives) [...]
      > Prose quality still needs a human eye.

      ## Name a thing the way the site names it

      One more rule ties tests and docs to the copy:

      > A story, test, or comment calls a thing what its label, message, or
      > column heading calls it. Copy the word a person reads in
      > `src/locales/en/*.json`, never the identifier beside it in `src/`.

      So a [Cucumber](https://cucumber.io/docs/) story says "the Concert has earned 45.00", not
      "`listing.income_total` equals 4500". The words in the specs, the
      words in the docs and the words on screen are the same words, and
      `check:e2e-labels` fails the build when the nightly tests would click
      a label that no longer exists.
---
