---
name: Glossary
title: Glossary
meta_title: Glossary | Chobble Coding Standards
meta_description: >-
  Plain explanations of the tools and ideas used on this site, for junior
  programmers: linters, duplication checks, test coverage, mutation
  testing, Cucumber, state machines, TLA+, Apalache, Nix and more.
permalink: /glossary/
eleventyNavigation:
  key: Glossary
  order: 4
blocks:
  - type: markdown
    content: |
      # Glossary

      This page explains the tools and ideas the rest of the site uses, in
      plain words. Each entry links to the official documentation, or to
      the best beginner introduction I know of. If a technique page uses a
      word you have not met before, it is probably here.

      **Jump to:** [Agents](#agents) · [Code quality tools](#code-quality) ·
      [Testing](#testing) · [Proofs](#proofs) ·
      [Tools and environment](#tools) · [Programming ideas](#ideas)

      <h2 id="agents">Working with LLM agents</h2>

      <h3 id="llm-agent">LLM coding agent</h3>

      A program built around a large language model (LLM) that can read
      files, run commands and edit code on its own, in a loop, until it
      thinks a task is done. Examples are
      [Claude Code](https://www.anthropic.com/claude-code),
      [OpenAI Codex](https://openai.com/codex/) and
      [OpenCode](https://opencode.ai/). This whole site is about keeping
      their output honest.

      <h3 id="agents-md">AGENTS.md</h3>

      A Markdown file at the root of a repository that tells coding agents
      how to work in it. [agents.md](https://agents.md/) describes the
      convention. Some tools look for their own file name instead (Claude
      Code reads `CLAUDE.md`), so those files can just point at AGENTS.md.

      <h3 id="ste">Simplified Technical English (STE)</h3>

      A controlled form of English with a limited vocabulary and strict
      grammar rules, first written for aircraft maintenance manuals so that
      instructions cannot be misread. See [ASD-STE100](https://www.asd-ste100.org/)
      and the [Wikipedia overview](https://en.wikipedia.org/wiki/Simplified_Technical_English).

      <h2 id="code-quality">Code quality tools</h2>

      <h3 id="linter">Linter</h3>

      A tool that reads source code without running it and reports
      patterns that are likely bugs or break the house style, such as an
      unused variable or `==` instead of `===`. Many linters can fix some
      problems for you. [What is a linter?](https://en.wikipedia.org/wiki/Lint_(software))

      <h3 id="biome">Biome</h3>

      A fast linter and code formatter for JavaScript and TypeScript, used
      by both example repositories. Its
      [rule list](https://biomejs.dev/linter/javascript/rules/) explains every rule with
      examples. Start with the
      [getting started guide](https://biomejs.dev/guides/getting-started/).

      <h3 id="cognitive-complexity">Cognitive complexity</h3>

      A score for how hard a function is to follow: each `if`, loop and
      level of nesting adds to it. A linter can fail any function above a
      limit. See Biome's
      [noExcessiveCognitiveComplexity](https://biomejs.dev/linter/rules/no-excessive-cognitive-complexity/)
      and the [original paper from SonarSource](https://www.sonarsource.com/docs/CognitiveComplexity.pdf).

      <h3 id="type-checker">Type checker</h3>

      The part of [TypeScript](https://www.typescriptlang.org/docs/handbook/intro.html)
      that checks your code uses values the way their types say, before the
      code runs. `deno check` and `tsc --noEmit` run it on their own. See
      [Strict type checking](/techniques/static-checks/strict-types/).

      <h3 id="jscpd">jscpd (copy-paste detector)</h3>

      A tool that finds duplicated blocks of code across a project by
      comparing runs of tokens (words and symbols). It can ignore names
      and values, so it also finds code that was copied and then renamed.
      See [jscpd on GitHub](https://github.com/kucherenko/jscpd).

      <h3 id="knip">Knip</h3>

      A tool that finds unused files, unused exports and unused
      dependencies in JavaScript and TypeScript projects, by following
      imports from your entry points. See [knip.dev](https://knip.dev/).

      <h3 id="dead-code">Dead code</h3>

      Code that nothing in the running program uses. It still has to be
      read, kept compiling and kept tested, so it costs time and hides real
      code. [Wikipedia](https://en.wikipedia.org/wiki/Dead_code).

      <h3 id="ratchet">Ratchet (or baseline)</h3>

      A committed list of the problems a check already knows about, which
      is allowed to get shorter but never longer. It lets you switch a new
      check on today without first fixing every old problem. See
      [Ratchets](/techniques/static-checks/ratchets/).

      <h3 id="import-cycle">Import cycle</h3>

      When file A imports file B and B (directly or through other files)
      imports A. In JavaScript this can leave a value undefined when one
      file starts running before the other has finished.
      [Wikipedia](https://en.wikipedia.org/wiki/Circular_dependency).

      <h2 id="testing">Testing</h2>

      <h3 id="unit-test">Unit test</h3>

      A small automated test of one function or module, with fixed inputs
      and checked outputs. [Martin Fowler's short definition](https://martinfowler.com/bliki/UnitTest.html).

      <h3 id="assertion">Assertion</h3>

      The line in a test that checks a result, such as
      `expect(total).toBe(45)`. A weak assertion like
      `expect(total).toBeDefined()` passes for almost any value. See the
      [expect API in Deno's standard library](https://jsr.io/@std/expect).

      <h3 id="coverage">Test coverage</h3>

      The share of your code's lines and branches (each side of an `if`)
      that ran at least once during the tests. It tells you what ran, not
      what was checked. [Wikipedia](https://en.wikipedia.org/wiki/Code_coverage).
      Coverage tools usually write an [lcov](https://github.com/linux-test-project/lcov)
      report that other tools can read.

      <h3 id="mutation-testing">Mutation testing</h3>

      A way to test your tests. A tool makes a small change to your code,
      such as turning `<` into `<=`, and runs the tests. If no test fails,
      the change "survived", and your tests have a gap. The
      [Stryker introduction](https://stryker-mutator.io/docs/) is the best
      beginner explanation, even though these projects use their own
      runners. [Wikipedia](https://en.wikipedia.org/wiki/Mutation_testing).

      <h3 id="equivalent-mutant">Equivalent mutant</h3>

      A mutation that does not change what the program does for any
      possible input, so no test can ever catch it. For example `x ?? "a"`
      and `x || "a"` behave the same when `x` can never be an empty string.

      <h3 id="cucumber">Cucumber and Gherkin</h3>

      [Cucumber](https://cucumber.io/docs/) runs tests
      written as plain-English scenarios. The scenario language is called
      [Gherkin](https://cucumber.io/docs/gherkin/reference/): `Given` some
      starting state, `When` something happens, `Then` this is the result.
      Each line is matched to a "step definition" in code that does the real
      work. This style is often called
      [behaviour-driven development](https://cucumber.io/docs/bdd/).

      <h3 id="e2e">End-to-end (e2e) test</h3>

      A test that drives the whole system the way a user would, often
      through a real browser using a tool like
      [Playwright](https://playwright.dev/docs/intro).

      <h3 id="mock">Mock and sandbox</h3>

      A **mock** is a fake version of an outside service that your tests
      talk to instead of the real thing, such as
      [stripe-mock](https://github.com/stripe/stripe-mock). A **sandbox** is
      a real test account at the provider, which behaves like production but
      moves no real money, such as
      [Stripe test mode](https://docs.stripe.com/testing) or the
      [Square sandbox](https://developer.squareup.com/docs/devtools/sandbox/overview).

      <h3 id="webhook">Webhook and callback</h3>

      A request that another service sends to your site to tell it
      something happened, such as "this payment succeeded".
      [Stripe's explanation](https://docs.stripe.com/webhooks) is a good
      introduction.

      <h3 id="idempotent">Idempotent</h3>

      An operation is idempotent when doing it twice has the same effect as
      doing it once. A payment webhook handler must be idempotent, because
      providers can send the same webhook more than once.
      [Stripe on idempotency](https://docs.stripe.com/api/idempotent_requests).

      <h3 id="race-condition">Race condition</h3>

      A bug that only happens when two things run at the same time in an
      unlucky order, such as two people booking the last seat at once.
      [Wikipedia](https://en.wikipedia.org/wiki/Race_condition).

      <h3 id="fault-injection">Fault injection</h3>

      Making part of the system fail on purpose during a test, to check the
      rest handles it. In these projects, an
      [SQLite trigger](https://www.sqlite.org/lang_createtrigger.html) makes
      one chosen database write fail.

      <h2 id="proofs">Machines and proofs</h2>

      <h3 id="state-machine">State machine</h3>

      A description of something that can be in one of a fixed set of
      states (such as `waiting`, `paid`, `refunded`) and the events that
      move it from one state to another.
      [Wikipedia](https://en.wikipedia.org/wiki/Finite-state_machine).
      [XState's introduction](https://stately.ai/docs/state-machines-and-statecharts)
      is friendlier.

      <h3 id="invariant">Invariant</h3>

      A rule that must always be true, such as "the number of booked seats
      is never more than the capacity".
      [Wikipedia](https://en.wikipedia.org/wiki/Invariant_(mathematics)#Invariants_in_computer_science).

      <h3 id="model-checking">Model checking</h3>

      Automatically checking every possible behaviour of a simplified model
      of a system against a rule, instead of only the cases someone wrote
      tests for. [Wikipedia](https://en.wikipedia.org/wiki/Model_checking).

      <h3 id="tla">TLA+</h3>

      A language for describing how a system changes over time, created by
      Leslie Lamport, so that tools can check it. Hillel Wayne's
      [Learn TLA+](https://learntla.com/) is the gentlest start. The
      [official site](https://lamport.azurewebsites.net/tla/tla.html) has
      Lamport's video course.

      <h3 id="apalache">Apalache</h3>

      A model checker for TLA+ that turns the question "can this rule ever
      break?" into a maths problem for an SMT solver. See the
      [Apalache manual](https://apalache-mc.org/docs/) and its
      [tutorial on symbolic model checking](https://apalache-mc.org/docs/tutorials/symbmc.html).

      <h3 id="smt">SMT solver</h3>

      A program that decides whether a set of logical and arithmetic
      conditions can all be true at once.
      [Wikipedia](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories).

      <h3 id="induction">Inductive invariant</h3>

      A rule that is true at the start, and stays true after any single
      step from any state where it was true. Those two facts together prove
      it is true forever, the same way
      [mathematical induction](https://en.wikipedia.org/wiki/Mathematical_induction)
      works.

      <h2 id="tools">Tools and environment</h2>

      <h3 id="deno">Deno</h3>

      A JavaScript and TypeScript runtime with a built-in test runner,
      formatter and type checker. The tickets repository runs on it.
      [Deno docs](https://docs.deno.com/runtime/).

      <h3 id="bun">Bun</h3>

      Another JavaScript runtime and package manager, used by the
      chobble-template and this site. [Bun docs](https://bun.sh/docs).

      <h3 id="bunny">Bunny Edge Scripting</h3>

      A service that runs JavaScript on [bunny.net](https://bunny.net/edge-scripting/)
      servers close to the user. Each request may start a fresh copy of the
      program (a "cold start"), which is why the tickets code avoids heavy
      work at startup.

      <h3 id="eleventy">Eleventy</h3>

      A static site generator: it turns Markdown and templates into plain
      HTML files. This site is built with it.
      [Eleventy docs](https://www.11ty.dev/docs/).

      <h3 id="nix">Nix and devenv</h3>

      [Nix](https://nixos.org/guides/how-nix-works/) is a package manager
      that can pin every tool to an exact version.
      [devenv](https://devenv.sh/getting-started/) builds a project's
      development environment on top of it, from one `devenv.nix` file.

      <h3 id="lockfile">Lockfile</h3>

      A file (such as `deno.lock`) that records the exact version of every
      dependency, so every install gets the same code.
      [Deno on lockfiles](https://docs.deno.com/runtime/fundamentals/modules/#integrity-checking-and-lock-files).

      <h3 id="ci">Continuous integration (CI)</h3>

      A service that runs your checks on every push.
      [GitHub Actions](https://docs.github.com/en/actions/get-started/understand-github-actions)
      is the one used here.

      <h3 id="git-hook">Pre-commit hook</h3>

      A script that git runs before it makes a commit, which can refuse the
      commit if a check fails.
      [Git hooks in the Git book](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks).

      <h3 id="stacked-prs">Stacked pull requests</h3>

      A large change split into a chain of small pull requests, each built
      on the one before it, so each can be reviewed on its own.
      [stacking.dev](https://www.stacking.dev/) explains the idea.

      <h2 id="ideas">Programming ideas</h2>

      <h3 id="pure-function">Pure function</h3>

      A function whose result depends only on its arguments, and which
      changes nothing outside itself: no database, no network, no clock.
      Pure functions are the easiest code to test.
      [Wikipedia](https://en.wikipedia.org/wiki/Pure_function).

      <h3 id="currying">Currying</h3>

      Turning a function that takes several arguments into a chain of
      functions that each take one. `add(1, 2)` becomes `add(1)(2)`, so you
      can make `addOne = add(1)` and reuse it.
      [javascript.info on currying](https://javascript.info/currying-partials).

      <h3 id="fp">Functional programming</h3>

      A style built from small pure functions combined with helpers like
      `map`, `filter` and `pipe`, instead of loops that change variables.
      [Mostly Adequate Guide](https://mostly-adequate.gitbook.io/mostly-adequate-guide/)
      is a free beginner book.

      <h3 id="schema">Schema validation</h3>

      Checking that data from outside your program (a form, an API
      response) has the shape you expect, once, at the point where it
      arrives. Both repositories use [Valibot](https://valibot.dev/guides/introduction/)
      for this.

      <h3 id="standard-library">Standard library</h3>

      The well-tested modules that ship with a language or runtime, or are
      kept alongside it. Deno's standard library lives at
      [jsr.io/@std](https://jsr.io/@std): `@std/collections` for lists and
      maps, `@std/assert` and `@std/expect` for tests, `@std/path` for file
      paths, and more. Using it instead of writing your own helper means
      less code to test. See
      [Libraries over hand-rolled code](/techniques/instructing-agents/standard-libraries/).

      <h3 id="discriminated-union">Discriminated union</h3>

      A TypeScript type that is one of several shapes, told apart by a
      shared field like `kind`. It lets each shape carry only the fields
      that make sense for it.
      [TypeScript handbook](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions).

      <h3 id="fail-fast">Fail fast</h3>

      Stopping with a clear error as soon as something is wrong, instead of
      carrying on with a guessed value.
      [Wikipedia](https://en.wikipedia.org/wiki/Fail-fast_system). The
      [Offensive programming](/techniques/static-checks/offensive-programming/)
      page shows it in practice.

      <h3 id="import-map">Import map and aliases</h3>

      A mapping from short names like `#fp` to real file paths, so imports
      do not need long `../../` paths. See
      [Deno's import maps](https://docs.deno.com/runtime/fundamentals/modules/#import-maps)
      and [Node.js subpath imports](https://nodejs.org/api/packages.html#subpath-imports).
---
