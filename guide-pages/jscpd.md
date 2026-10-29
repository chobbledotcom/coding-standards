---
name: jscpd and its variants
subtitle: Copy-paste detection at a 0% threshold, eight scans deep
guide-category: static-checks
order: 3
meta_description: >-
  How the tickets repository runs jscpd at a 0% duplication threshold:
  per-tree token limits, the ignoreIdentifiers, ignoreLiterals and
  ignoreAnnotations flags, a similarity gate for renamed copies, a TLA+ adapter, and the rule that a flagged pair is one
  merge.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      [jscpd](/glossary/#jscpd) is a copy-paste detector. It splits code into
      *tokens* (names, keywords, symbols) and looks for long runs of tokens that
      appear more than once. Read the
      [jscpd README](https://github.com/kucherenko/jscpd#readme)
      for its options. The principle behind it is often called
      [DRY: Don't Repeat Yourself](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself).
  - type: markdown
    content: |
      ## The agent's favourite shortcut

      Copying is the cheapest way for an agent to add behaviour. Find a
      function that nearly does the job, paste it, change one value. Each
      copy is correct on the day it lands. Six months later a bug is fixed
      in one copy and not the other four.

      [jscpd](https://github.com/kucherenko/jscpd) finds repeated runs of
      tokens across a codebase. The tickets repository runs it with a
      threshold of **0%**: any clone at all fails the build. AGENTS.md calls
      the threshold "non-negotiable".

      ## The real knob is `minTokens`

      A 0% threshold sounds absolute, but what decides how hard jscpd looks
      is `minTokens`: the shortest run of tokens that counts as a clone. The
      tickets scans set it low: 19 tokens for production code.

      ## The `ignore` flags: seeing through renames

      jscpd 5 has three options that change *what* it compares.
      [#2866](https://github.com/chobbledotcom/tickets/pull/2866) turned
      them on in the four strict scans:

      - `ignoreIdentifiers` treats every variable and function name as the
        same token, so `total` and `sum` match.
      - `ignoreLiterals` does the same for strings and numbers, so `"admin"`
        and `"owner"` match.
      - `ignoreAnnotations` does the same for type annotations.

      With all three on, jscpd finds blocks that have the same *shape* even
      when every name and value is different. That is the "paste and
      rename" copy an agent makes. jscpd then sorts each match it finds
      into a **kind**: an `exact` clone has identical raw tokens, and a
      `renamed` clone differs only in names, values or types.

      The strict scans add `"kind": ["exact"]`, so they still fail only on
      code that is token-for-token the same. The renamed matches go to the
      renamed-clone gate, described below. The flags still
      catch more, because jscpd splits a long renamed match at the tokens
      that differ. If a renamed copy leaves an identical stretch of
      `minTokens` or more, the strict gate reports that stretch and asks
      for the merge.

      ```json
      {
        "threshold": 0,
        "ignoreIdentifiers": true,
        "ignoreLiterals": true,
        "ignoreAnnotations": true,
        "kind": ["exact"],
        "minLines": 1,
        "minTokens": 19,
        "path": ["src", "e2e-payments", "scripts"]
      }
      ```

      The four configs repeat this block by hand, so a small test,
      [`test/scripts/jscpd/config.test.ts`](https://github.com/chobbledotcom/tickets/blob/main/test/scripts/jscpd/config.test.ts),
      fails when one of them drifts. A check on a check sounds excessive,
      but it costs ten lines and stops one config quietly losing a flag.

      ## Eight scans, each tuned to its tree

      One threshold does not fit a whole repository. Test setup repeats more
      than production code, and CSS repeats short declaration runs by
      design. So [`deno task cpd`](https://github.com/chobbledotcom/tickets/blob/main/scripts/cpd.ts) runs several scans, each with its own
      config, and
      [docs/test-duplication.md](https://github.com/chobbledotcom/tickets/blob/main/docs/test-duplication.md)
      records what each step down would cost.

      | Config | Scans | minTokens |
      | --- | --- | --- |
      | [`.jscpd.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.json) | `src`, `e2e-payments`, `scripts` | 19 |
      | [`.jscpd.specs.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.specs.json) | `src` + Cucumber support | 18, with a baseline |
      | [`.jscpd.support.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.support.json) | Cucumber support alone | 15 |
      | [`.jscpd.helpers.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.helpers.json) | `src` + `test/test-utils` | 40 |
      | [`.jscpd.test.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.test.json) | `test` | 48 |
      | [`.jscpd.css.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.css.json) | the main stylesheets | 50 |
      | [`.jscpd.renamed.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.renamed.json) | `src`, `e2e-payments`, `scripts` | 20, 99% similarity, with a baseline |
      | [`cpd:tla`](https://github.com/chobbledotcom/tickets/blob/main/scripts/tla-cpd.ts) | every TLA+ model | own lexer |

      Scanning `src` *together with* the test helpers matters. It catches a
      test helper that re-implements a production function instead of
      importing it.

      ## The renamed-clone gate

      The strict scans report only `exact` clones, so a function that was
      copied and renamed in full slips past them. The renamed scan runs
      jscpd's `--similarity 0.99` gate, which compares functions by the
      shape of their syntax tree with names and values left out. A renamed
      copy scores 1.0. Existing matches live in [`.jscpd.renamed-baseline.json`](https://github.com/chobbledotcom/tickets/blob/main/.jscpd.renamed-baseline.json) as
      fingerprints, and `--fail-on-new-clones` fails on any new one. This
      replaced two hand-rolled scanners
      ([#2380](https://github.com/chobbledotcom/tickets/pull/2380)), which
      is its own lesson: when the tool grows the feature, delete your
      version.

      ## Duplication in proofs

      The [TLA+ models](/techniques/machines-and-proofs/tla-and-apalache/)
      are code too, and they get copy-pasted too.
      [`scripts/tla-cpd.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/tla-cpd.ts)
      lexes each `.tla` file into a form jscpd can read, keeping
      identifiers, literals and prime marks, and dropping comments. Shared
      rules move into [`verification/ModelRules.tla`](https://github.com/chobbledotcom/tickets/blob/main/verification/ModelRules.tla).

      ## A flagged pair is one merge

      The hardest part of duplication detection is not the tool, it is what
      people (and agents) do when it fires. AGENTS.md is explicit:

      > - Write the helper, or give the shared tail its parameter. Two
      >   functions that differ only in a value, a path, a field name, a
      >   message, or a callback are one function without its parameter
      >   yet.
      > - Never restructure code so the matcher stops matching while the
      >   two parallel implementations stay standing.
      > - After a dedup, search for the other places that now fold into the
      >   new helper.

      The second bullet is the important one. An agent can always reorder
      a few statements until jscpd stops matching. That passes the check
      and keeps the problem. The rule names that move and forbids it.

      The only sanctioned repeat is import blocks, wrapped in
      `jscpd:ignore-start` / `jscpd:ignore-end` markers. Everything else is
      a merge.

  - type: split-code
    subtitle: Guidance at the point of failure
    content: |
      ## The wrapper that tells you what to do

      jscpd reports *where* the clone is. The repository's wrapper,
      [`scripts/cpd.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/cpd.ts),
      runs jscpd unchanged and, only on failure, prints the house rule:
      extract a helper or curry, and keep `jscpd:ignore` for imports.

      An agent reads error output much more carefully than documentation.
      Put the instruction where the failure is.
    figure_filename: scripts/cpd.ts
    figure_language: typescript
    figure_code: |
      /**
       * jscpd runner that prints actionable guidance
       * when duplication is found.
       *
       * jscpd's own console report tells you *where*
       * the duplication is, but not what to do about
       * it. This wrapper runs jscpd unchanged and
       * forwards every arg, then — only on a non-zero
       * exit — appends a loud reminder of the
       * project's non-negotiable 0% policy: extract a
       * helper, or curry, and reserve `jscpd:ignore`
       * for the one thing it is actually for
       * (import blocks).
       */

  - type: markdown
    content: |
      ## Practical notes

      - **Pin the binary.** jscpd 5 is a Rust binary shipped through npm.
        The tickets repository downloads the static musl build and pins its
        SHA-256
        ([`scripts/jscpd/install.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/jscpd/install.ts)),
        because the glibc build cannot start on [NixOS](https://nixos.org/).
      - **Currying is the usual fix.** Two functions that differ in a value
        become one factory that takes the value and returns the function.
        That is why the codebase is curried throughout.
      - **The chobble-template ratchets the threshold.** Its [`cpd:ratchet`](https://github.com/chobbledotcom/chobble-template/blob/main/scripts/cpd-ratchet.js)
        task fails when the code would pass at one token lower, so the
        threshold only tightens. See [Ratchets](/techniques/static-checks/ratchets/).
---
