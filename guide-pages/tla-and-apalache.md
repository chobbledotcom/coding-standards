---
name: TLA+ models checked by Apalache
subtitle: Prove the safety rules hold for runs of any length, using the real transitions
guide-category: machines-and-proofs
order: 2
meta_description: >-
  How the tickets repository generates TLA+ model data from its production
  state machines and checks safety with the Apalache model checker: an
  initial-state check, an inductive step, a deliberate counterexample, and
  honest limits.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Why tests are not enough here

      The [machine sweep](/techniques/machines-and-proofs/state-machines/)
      checks every single step. It does not check every *sequence* of
      steps. A rule like "a refund request is never sent twice" is about
      histories: a send, a crash, a lease expiring, a second worker, a
      replayed callback. Tests check the histories someone wrote down. A
      model checker checks all of them.

      [TLA+](https://lamport.azurewebsites.net/tla/tla.html) is Leslie
      Lamport's language for describing systems as states and next-state
      relations. [Apalache](https://apalache-mc.org/) is a symbolic model
      checker for TLA+ that uses an SMT solver, which lets it check an
      *inductive* invariant: a property that, if it holds in any state,
      still holds after any step.

      ## The models

      The tickets repository has its models in
      [`verification/`](https://github.com/chobbledotcom/tickets/tree/main/verification):

      | Model | What it proves |
      | --- | --- |
      | `JointState.tla` | The refund money edges: only a money edge can arm a send, conclusive "not sent" evidence never re-arms one, a keyless request never replays automatically, and a refund row is deleted only once the refund is complete and recorded. |
      | `SumupRecovery.tla` | Records that might hold unaccounted money cannot be pruned, always keep a check path, and paid or terminal evidence never moves backwards. |
      | `PaymentReview.tla` | Each review state keeps its reason, acknowledgement preserves the case, only matching evidence retires it, and review events never move money. |
      | `SquareLinkEnd.tla` | Cancel handles are kept while a link can still take payment, and inconclusive replies keep their retry work. |
      | `TwoWorkers.tla` | A worker that fell behind can never release or overwrite the claim of the worker that replaced it, and lease expiry never cancels a request already in flight. |
      | `BookingReplay.tla` | One payment session delivered more than once: once the durable ledger records the booking as honoured, no delivery books again and none requests a refund, whatever happened to the temporary row. |
      | `VatLedger.tla` | Every assess, reassess and refund attempt keeps its VAT basis, setting, revision and arithmetic correct, and none calls a payment provider. |

      ## No hand-copied tables

      The important design choice: the model never contains a hand-written
      copy of the move tables. `deno task verify:models` *executes the
      production transitions*, collects what they actually do, and writes
      that out as a TLA+ data module for each model
      ([`scripts/verify-models/`](https://github.com/chobbledotcom/tickets/tree/main/scripts/verify-models)).
      The `.tla` file holds only the rules.

      So the proof is about the code that ships, not about someone's
      memory of it. Change a transition and the next `verify:models` run
      checks the new behaviour.

  - type: split-code
    subtitle: Reading a model
    content: |
      ## The safety property, in its own words

      The SumUp model states the money rule directly. A row in a risky
      state must be retained, and must still have an enabled check.

      Pruning in the model is deliberately naive: it uses only age and the
      declared prunable flag, exactly like production. If the declarations
      were wrong, `riskyDeletionSeen` would become true and `Safety` would
      fail.
    figure_filename: verification/SumupRecovery.tla
    figure_language: tla
    figure_code: |
      RiskStates == MoneyStates("unknown") \cup MoneyStates("yes")

      RiskyMoneyKeepsQueuedWork ==
        /\ (recoveryState \in RiskStates => rowRetained)
        /\ (rowRetained /\ recoveryState \in RiskStates =>
             /\ recoveryState \in CheckableStates
             /\ HasEnabledCheck(recoveryState, hasCheckoutId))

      \* Deletion uses only age and the declared
      \* pruning state, not money safety.
      Prune ==
        /\ rowRetained
        /\ pruningAgeReached
        /\ recoveryState \in PrunableStates
        /\ rowRetained' = FALSE
        /\ riskyDeletionSeen' =
             (riskyDeletionSeen \/ recoveryState \in RiskStates)
        /\ UNCHANGED <<...>>

  - type: markdown
    content: |
      ## Two checks per model: start, and every step

      Each model is checked twice
      ([`scripts/verify-models.ts`](https://github.com/chobbledotcom/tickets/blob/main/scripts/verify-models.ts)):

      ```typescript
      const checks = [
        { init: "Init", length: 0, name: "initial state" },
        { init: "InductionStart", length: 1, name: "every legal state and next edge" },
      ];
      // apalache-mc check --init=<init> --next=Next --inv=Safety --length=<n> <Model>.tla
      ```

      1. `Init` with length 0: the starting state is safe.
      2. `InductionStart` with length 1, where `InductionStart == Safety`:
         from *any* state that satisfies `Safety`, *any* single step lands
         in a state that still satisfies `Safety`.

      Together those two are an induction proof. The property holds at the
      start, and no step can break it, so it holds after any number of
      steps. This is why the design guide can say the checks "establish
      model safety for any run length", rather than up to some bound.

      ## Make the checker prove it can fail

      A model checker that says "no errors" is only useful if it would have
      said something else when there was an error. A broken data
      generator, a typo in an invariant, or an always-false guard can all
      make every check pass. So `verify:models` corrupts the data on
      purpose and requires a counterexample:

      ```typescript
      // The checker must reject corrupt data. A pass here would make the
      // clean verdicts worthless.
      console.log("Check JointState: corrupted edge counterexample.");
      // ...writes JointStateData.tla from corruptOneEdge(model)...
      const corrupted = await runCheck("JointState", "InductionStart", 1);
      if (corrupted.success) {
        throw new Error(
          "The checker accepted a corrupted edge. The safety verdict is not trusted.",
        );
      }
      ```

      `corruptOneEdge` takes one real money edge and flips the fact that it
      keeps its refund request. `BookingReplay` gets the same treatment for
      both failure modes it rules out: a duplicate booking, and a refund
      after the temporary row was deleted. This is mutation testing for
      proofs.

      ## Proofs are code, so treat them like code

      - **Readable rules.** "Give each fact named record fields. Use short
        rule names that describe the business condition. Prefer explicit
        conditions to dense Boolean formulas." `RiskyMoneyKeepsQueuedWork`
        is a sentence an operator could read.
      - **Shared helpers.** Common rules live in
        [`ModelRules.tla`](https://github.com/chobbledotcom/tickets/blob/main/verification/ModelRules.tla),
        for example `ChecksStillPass`: "A later success cannot erase an
        earlier failure."
      - **Duplication checked.** `deno task cpd` includes a TLA+ adapter that
        runs jscpd over every model.
      - **Types beside declarations.** Apalache type notes sit next to each
        variable, and each fact's schema is declared once in its generator,
        from which both the TypeScript type and the TLA+ type note derive.
      - **Pinned tools.** Apalache and its Java runtime come from devenv,
        so the model check runs the same in precommit and CI.

      ## Say what the proof does not prove

      The design guide lists the limits, and that list matters as much as
      the proofs:

      > The models trust representative coverage and event annotations. [...]
      > Model safety does not prove SQL atomicity, provider truth or eventual
      > task execution.

      Each of those gaps has its own check: the
      [conformance sweep](/techniques/machines-and-proofs/state-machines/)
      for representatives, database tests for atomicity, the
      [nightly sandbox runs](/techniques/testing/real-endpoints/) for
      provider truth, and the live anomaly scan for stored data. A proof
      that claims to cover everything is a proof nobody has read closely.

  - type: callout
    variant: info
    icon: "mdi:history"
    name: How it replaced a table
    content: |
      The first Apalache model
      ([#2737](https://github.com/chobbledotcom/tickets/pull/2737))
      replaced a hand-maintained table of "illegal joint states" and the
      runtime guards and scans that depended on it. The proof covered the
      same two impossible states for every reachable run, so the table and
      everything only it could feed was deleted, and two regression tests
      kept the original bugs pinned.
---
