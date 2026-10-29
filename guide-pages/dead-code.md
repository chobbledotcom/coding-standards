---
name: Knip and dead code
subtitle: Unused files, unused exports, test-only exports and fields nobody reads
guide-category: static-checks
order: 5
meta_description: >-
  How dead code is found and removed: knip in the chobble-template, a
  find-unused-src scan in tickets, a test-only-exports check, and a
  type-checker-based scan for exported fields that no production code
  reads.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      [Dead code](/glossary/#dead-code) is code nothing uses. [Knip](/glossary/#knip) finds it
      by starting from your *entry points* (the files your program starts
      from) and following every import. Anything it never reaches is unused.
      The [Knip getting started guide](https://knip.dev/overview/getting-started)
      explains entry points well. The unread-fields scan uses the
      [TypeScript compiler API](https://github.com/microsoft/TypeScript/wiki/Using-the-Compiler-API).
  - type: markdown
    content: |
      ## "Remove dead code — always the answer"

      Agents are reluctant to delete. They keep the old function "for
      backwards compatibility", leave an export "in case it is useful", and
      add a test so the unused code is at least covered. AGENTS.md answers
      all three at once:

      > When code has no production caller, delete it. Never keep it "for
      > symmetry" or "for future use", and never paper over it with a
      > test-only import or a usage-check exemption. An export that only
      > tests use is dead.

      The last sentence is the subtle one. An export that only a test
      imports *looks* used to most tools. It is not. Several checks exist to
      find each kind of dead code.

      ## Knip, in the chobble-template

      [Knip](https://knip.dev/) builds the module graph from a list of
      entry points and reports files, exports and dependencies that nothing
      reaches. The template runs it as a precommit step with a read-only
      [`knip`](https://github.com/chobbledotcom/chobble-template/blob/main/package.json), and offers [`knip:fix`](https://github.com/chobbledotcom/chobble-template/blob/main/package.json) to apply removals.

      The interesting part of the config is the entry list. An [Eleventy](https://www.11ty.dev/docs/)
      site has many implicit entry points (data files, `.11tydata.js`
      files, browser bundles), and every one has to be named, or knip
      reports live code as dead:

      ```json
      {
        "entry": [
          ".eleventy.js",
          "src/_data/**/*.{js,mjs}",
          "src/**/*.11tydata.js",
          "src/_lib/public/bundle.js",
          "test/**/*.test.js",
          "packages/js-toolkit/fp/index.js"
        ],
        "project": ["**/*.{js,mjs}"],
        "ignoreExportsUsedInFile": true
      }
      ```

      See the full
      [`knip.json`](https://github.com/chobbledotcom/chobble-template/blob/main/knip.json).
      Note that test files are entries, so knip alone would treat a
      test-only export as alive. That is what the next check is for.

      ## Test-only exports

      The template's
      [`test-only-exports.test.js`](https://github.com/chobbledotcom/chobble-template/blob/main/test/unit/code-quality/test-only-exports.test.js)
      finds exports from `src/` that only `test/` imports:

      > When a function is exported but only ever imported in test/, it
      > suggests the tests are testing implementation details rather than
      > the public API.

      In tickets, the same job is done by
      [`scripts/find-unused-src.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/find-unused-src.ts),
      which runs three checks over the import map: `src/` files that only
      tests import, individual exports that only tests use, and `src/` files
      nothing imports at all. It follows static imports, dynamic
      `await import(...)` calls and the `#` aliases in [`deno.json`](https://github.com/chobbledotcom/tickets/blob/main/deno.json).

      ## Fields that nothing reads

      Unused *exports* are the easy case. Unused *fields* are harder, and
      they cause real bugs. The motivating case, from
      [the scan's README](https://github.com/chobbledotcom/tickets/blob/main/scripts/unread-fields/README.md):

      > `BulkSendResult.failed` counted a bulk email send's refused
      > recipients. No production code read it, so the operator was told
      > every message went.

      A text search cannot find this. `.failed` appears on other types, on a
      webhook result, and inside a translation key, so a name match calls
      the dead field alive. So the scan asks TypeScript. It builds a
      program over the repository with the same compiler options the
      repository uses, walks it once, and asks, symbol by symbol, who reads
      each exported field.

      The compiler options matter more than they look. Under `strict`, an
      `if (!result.ok)` narrows a result to its failure arm, and the read of
      `result.error` belongs to that arm's field. Under the wrong options,
      46 fields that production does read were reported as dead.

      ## Holding the line on unread fields

      [`deno task check:unread-fields`](https://github.com/chobbledotcom/tickets/blob/main/scripts/check-unread-fields.ts) runs in precommit as a ratchet with
      two registries:

      - `baseline.ts` is accepted debt. "An entry does not say that the
        field is valid. Do not add new entries."
      - `exemptions.ts` holds reviewed false positives. Each entry must name
        its production evidence and one of five mechanisms: an external
        output, a provider input, a schema-driven read, a persisted format,
        or a dynamic read.

      The gate fails on a new field with no policy, on a policy entry that
      no longer matches a finding, and on a field in both registries. A
      finite exported type uses a closed snapshot, so adding a field to it
      stops typecheck until someone decides whether the field is read.

  - type: callout
    variant: info
    icon: "mdi:lightbulb-on-outline"
    name: Why the type checker beats grep
    content: |
      Any dead-code tool that matches names will be fooled by common names.
      Fields called `error`, `failed`, `id` or `status` exist on dozens of
      types. Asking the compiler which symbol each mention resolves to is
      slower to build, but it is the only answer you can gate on.
---
