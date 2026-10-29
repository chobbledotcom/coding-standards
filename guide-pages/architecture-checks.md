---
name: Architecture checks
subtitle: A pure core, no import cycles, one name per module, and short files
guide-category: static-checks
order: 6
meta_description: >-
  Checks that hold the shape of the codebase: an effect table that keeps the
  pure core away from the database, network and clock, a zero-cycle import
  gate, one import name per module, no alias exports, and file-length limits.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      These checks look at how files import each other. Words on this page:
      [pure function](/glossary/#pure-function), [import cycle](/glossary/#import-cycle) and
      [import map](/glossary/#import-map). An *effect* is anything a function does
      besides return a value: read the database, call the network, read the
      clock. Keeping effects at the edges is sometimes called
      ["functional core, imperative shell"](https://www.destroyallsoftware.com/screencasts/catalog/functional-core-imperative-shell).
  - type: markdown
    content: |
      Lint looks at one line at a time. These checks look at how files fit
      together. They exist because architecture erodes one import at a
      time, and an agent adding one import to finish a task never sees the
      ring it closes.

      ## Keep the pure core pure

      The design guide asks for a pure core with a thin IO shell around it.
      That is easy to say and easy to break: one import of a module that
      imports the database client, three files away, and a "pure" money
      rule is now impure.

      [`deno task check:effects`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-effects.ts) makes the split mechanical.
      [`scripts/check-effects/effects.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-effects/effects.ts)
      declares the effect roots and the pure list as data:

      ```typescript
      export type EffectName = "clock" | "db" | "net";

      export const EFFECT_ROOTS: Readonly<Record<EffectName, readonly string[]>> = {
        clock: ["#shared/now.ts"],
        db: ["#db/client.ts", "#db/database-client.ts"],
        net: [
          "#db/database-client.ts",
          "#shared/fetch.ts",
          "#shared/stripe.ts",
          // ...
        ],
      };

      /** The pure core: modules that must not reach any effect root. */
      export const PURE_MODULES: readonly string[] = [
        "#accounting/accounts.ts",
        // ...
      ];
      ```

      The check walks every import, static or dynamic, from each pure
      module. A finding prints the chain of imports that reaches the
      boundary, so the fix is obvious: split the IO shell out of the pure
      module, or, if the module really has become IO, take it off the pure
      list in a visible change.

      Reading the wall clock counts as an effect. A function that calls
      `Date.now()` cannot be tested without faking time, so the clock is a
      boundary like the database.

      ## No import cycles

      Import cycles cause the worst kind of startup bug: "Cannot access X
      before initialization", in one entry point and not another. The
      design guide describes one that passed the whole [Deno](https://docs.deno.com/runtime/) suite and took
      down every [Cucumber](https://cucumber.io/docs/) Feature.

      [`test/integration/import-cycles.test.ts`](https://github.com/chobbledotcom/tickets/blob/main/test/integration/import-cycles.test.ts)
      held `src/` to the cycles it already had, and that list only shrank.
      The rings were broken one at a time. When the count reached zero,
      [`deno task cycles`](https://github.com/chobbledotcom/tickets/blob/main/scripts/cycles.ts) became a hard precommit gate.

      ## Imports name a module one way

      [`deno task check:imports`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-imports.ts) enforces two things: a file imports each
      module once, and it uses the shortest alias the import map gives that
      module. That sounds cosmetic. It makes every search for a module's
      callers find all of them, and it stops the slow sprawl of aliases.
      AGENTS.md even sets a bar for new aliases: add one "only when it
      removes 50 or more wrapped import lines".

      ## No alias exports

      > Never export a name that only renames an imported method. Expose the
      > underlying helper and let callers use it.

      Agents create alias exports to avoid touching callers:
      `export const getThing = fetchThing;`. Now there are two names for
      one function, and half the codebase uses each.
      [`deno task check:alias-exports`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-alias-exports.ts) fails on any export that only renames
      an import. A thin wrapper that adds a default, a transformation or a
      guard is fine.

      The same rule shows up as **No internal compatibility layers**: when
      an internal API changes, migrate every caller in the same change and
      delete the old surface.

      ## Short files

      > The aim is 400 lines. The enforced limit is 500. Biome holds a hard
      > 1,000-line ceiling.

      [`deno task check:file-lengths`](https://github.com/chobbledotcom/tickets/blob/main/scripts/counts-check.ts) enforces the 500-line limit against a
      [ratchet](/techniques/static-checks/ratchets/) of files that were
      already over. Three numbers, three purposes: 400 is the target an
      agent aims for, 500 is the gate, and 1,000 is the backstop that holds
      even if the ratchet file is edited.

      There is guidance on splitting, too: give the new files shorter
      names that do not repeat the folder name, and separate pure
      data-in/data-out logic from the IO shell. And when a small edit pushes
      a file a little over 400, the rule is to open (or update) an issue
      that owns the split, not to split it in an unrelated pull request.

  - type: callout
    variant: info
    icon: "mdi:lightbulb-on-outline"
    name: Make the architecture data
    content: |
      The effect table and the pure list are plain arrays in a [TypeScript](https://www.typescriptlang.org/docs/)
      file. Moving a module on or off the pure list is a one-line diff a
      reviewer can see. When architecture rules live in a diagram or a
      wiki, nobody notices them change.
---
