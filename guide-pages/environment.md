---
name: A pinned environment
subtitle: Nix and devenv pin every tool, so a pass means the same thing everywhere
guide-category: workflow
order: 5
meta_description: >-
  How the tickets repository pins its toolchain with Nix and devenv: the
  lowest supported Deno version, frozen dependency installs with a
  lifecycle-script allowlist, checksummed binaries, and CI that runs inside
  the same shell.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Why pin everything

      A check is only useful if it gives the same answer everywhere. When
      lint passes locally and fails in CI because the linter versions
      differ, everyone (agents included) learns that red does not mean
      broken. Every tool in the tickets repository is pinned, and the pins
      are checked.

      ## devenv, on Nix

      The developer environment is [devenv](https://devenv.sh/)
      (`devenv.nix`), built on pinned `nixpkgs` commits in `devenv.yaml`.
      It provides Deno, Biome, Chromium for browser tests, Apalache and its
      Java runtime for the model checks, and the other tools. One command
      runs anything inside it:

      ```bash
      devenv shell deno task precommit
      devenv shell -- deno task test --filter "formats date"
      ```

      CI uses the same environment through a shared setup action, with a
      lighter `ci` profile for jobs that never launch a browser.

      ## Pin the lowest version you support

      The production runtime is Bunny Edge Scripting, which is Deno-based.
      The repository pins Deno **2.5.6**, "the lowest Bunny Edge Scripting
      runtime version this project runs on", not the newest. Testing on the
      oldest supported runtime means a feature from a newer Deno cannot
      sneak in and pass locally.

      The pin is enforced: `enterShell` in `devenv.nix` refuses to enter
      when the pinned nixpkgs provides any other Deno version. Upgrading
      the pin is a deliberate edit of a commit hash, followed by
      `devenv update`.

      ## Frozen installs, with an allowlist

      CI and the container image install dependencies with one script,
      [`scripts/ci/deno-install.sh`](https://github.com/chobbledotcom/tickets/blob/main/scripts/ci/deno-install.sh):

      ```bash
      deno install \
        --frozen \
        --allow-scripts=npm:@parcel/watcher,npm:esbuild,npm:fsevents,npm:sharp
      ```

      - `--frozen` fails on any dependency change that did not update
        `deno.lock`, instead of quietly resolving a new tree.
      - The lifecycle-script allowlist must exactly match the packages
        `deno.lock` records with `"scripts": true`. A test enforces the
        match, so a new dependency that wants to run an install script is a
        visible decision.

      ## Checksummed binaries

      Tools that come from outside Nix are downloaded once into `.bin/` and
      pinned:

      - **jscpd** is fetched as the static musl build and checked against a
        pinned SHA-256, because the glibc build cannot start on NixOS.
      - **stripe-mock** is pinned to an exact version (`v0.188.0`). The test
        harness starts one per run on a free port, so parallel suites never
        fight over the default port.
      - The **devenv CLI** in CI is pinned to a source revision rather than
        the moving `nixpkgs#devenv`.

      ## Agents and the environment

      Two rules in AGENTS.md follow from all this:

      - Non-interactive agents must prefix every `deno` command with
        `devenv shell`, so they always run the pinned tools.
      - The environment is infrastructure. If it breaks, the agent
        [stops and reports](/techniques/instructing-agents/guardrails/) the
        exact command and error. It does not install a different Deno, edit
        `devenv.nix`, or work around it.

      This site is built the same way: the
      [chobble-client](https://git.chobble.com/chobble/chobble-client)
      template it uses ships its own `devenv.nix` with a pinned Bun.
---
