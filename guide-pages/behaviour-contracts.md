---
name: Behaviour contracts
subtitle: Plan the states, failures, retries and races before any code exists
guide-category: instructing-agents
order: 2
meta_description: >-
  The pull request workflow from the tickets repository: a behaviour
  contract with trusted facts, valid states, failure, retry and concurrency
  tables, an adversarial self-review and human approval before code.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Why a plan comes first

      Left to itself, an agent starts typing. It discovers the behaviour by
      writing code, then discovers the edge cases through review, then
      patches each one where it was found. After three rounds the code has
      guards in four layers and nobody can say what the rules are.

      [PR_WORKFLOW.md](https://github.com/chobbledotcom/tickets/blob/main/PR_WORKFLOW.md)
      reverses that. It opens:

      > Use this workflow for every non-trivial feature, bug fix, or
      > refactor. The work must happen in this order. Do not start with
      > implementation and discover the behavior through review.

      ## Three problems, three tools

      The workflow separates three things that are often blurred together:

      1. **Schemas describe facts.** They say what a value is and which
         facts must travel together. A completed payment must carry the
         payment id, amount, currency, account, parent and proof.
      2. **State transitions describe changes.** They say which command can
         move a record from one valid state to another.
      3. **Transactions or revision checks protect races.** They stop two
         valid requests, or one stale one, producing the wrong final state.

      > A strict schema cannot stop a stale form overwriting a newer choice.
      > A transaction cannot make incomplete provider facts complete. Design
      > all three.

      ## State the value first

      Every plan opens with one sentence naming what gets better *if no
      later pull request ships*, and the production route that receives the
      change. The workflow gives a bad and a good example:

      > Bad: Add payment resource schemas for future use.
      >
      > Good: Current Square callbacks reject a charge whose currency
      > differs from the signed checkout currency.

      This one rule kills the most common agent plan: a "foundation" pull
      request of types and helpers with no caller, followed by a second pull
      request that never quite arrives.

      ## The contract sections

      The plan fills in a fixed set of sections. A section that does not
      apply says `None`; it is never left out.

      - **Trusted facts.** Every input, whether it is trusted, and why.
        Expected facts (what the signed checkout says) stay separate from
        observed facts (what the provider reports). "Never substitute an
        expected fact for a missing observed fact."
      - **Valid states.** Every state and the facts it requires, as a
        discriminated union where variants carry different facts. "An
        unavailable read is not unpaid. Missing data is not an empty value."
      - **Commands and events.** Every action that can change state,
        including callbacks, scheduled work, retries, migrations and admin
        repairs, with the exact next state.
      - **Failure table.** Failures at every boundary, what work was already
        done, the required result, and *who owns the retry*.
      - **Retry and replay table.** The stable identity that makes each
        command idempotent, what an exact replay returns, and what stops two
        workers doing the same job.
      - **Concurrency table.** Every pair of operations that can overlap,
        and the protection (an expected revision, a claim, a unique key).
      - **Owner choices.** Real conflicts the system cannot decide safely,
        surfaced to the operator as a required choice.
      - **Security and privacy.** Who can do each action, which links each
        role sees, and which secrets cross which boundary.

  - type: split-code
    subtitle: Make impossible states unrepresentable
    content: |
      ## Prefer a union to optional fields

      An optional field that means different things in different states is
      a bug waiting for a reader. The workflow asks for a union where each
      variant carries exactly its own facts.
    figure_filename: PR_WORKFLOW.md
    figure_language: typescript
    figure_code: |
      type Result =
        | { kind: "completed"; payment: CompletedPayment }
        | { kind: "pending"; paymentId: string }
        | { kind: "failed"; reason: FailureReason }
        | { kind: "unavailable"; retryAt: string };

  - type: markdown
    content: |
      ## Declare the machine once

      When a feature has a lifecycle, the tables above are not written by
      hand. Separate tables drift, silently, because nothing compares them.
      Instead the plan declares the
      [state machine](/techniques/machines-and-proofs/state-machines/) as
      data, and each table becomes a projection of that one declaration.
      The plan's invariants are written as laws over the declaration, so
      each one is a test rather than a sentence.

      ## Challenge it, then ask a human

      Before presenting the plan, the agent must try to break it. The
      workflow lists the minimum questions, including:

      - What if the external call succeeds and the local write fails?
      - What if the callback is replayed?
      - What if two requests run together?
      - What if one request uses a stale form?
      - What if the user reloads after an interruption?
      - What does the buyer or operator see in every unfinished state?

      Then it stops. No tests, no code, until a person explicitly approves
      this version of the plan:

      > Silence, a previous broad goal, or approval of an earlier draft is
      > not approval of the current contract.

      ## Vertical slices

      Pull requests are split by complete behaviour, never by layer. Good
      slices from the document: "Every provider rejects blank resource
      ids." "A failed provider read can never become an unpaid result." Bad
      slices: "Add types." "Add repositories." "Add one provider while
      keeping a parallel implementation for the others."

      ## When to stop and redesign

      The workflow names the signals that a design is wrong, and says to go
      back to the contract instead of adding another guard:

      - review finds two cross-cutting state or concurrency bugs;
      - two routes need slightly different versions of the same rule;
      - a new optional field changes meaning by state;
      - the same failure is handled in more than one layer;
      - tests need many special cases to set up one state.

      Agents are very good at patching. This list tells them when patching
      has become the problem.

  - type: callout
    variant: info
    icon: "mdi:clipboard-check-outline"
    name: Ready to code
    content: |
      Implementation starts only when the value and production caller are
      named, trusted and observed facts are separate, every state, command,
      failure, retry and race has an answer, the adversarial review has no
      open question, the tests that will prove each row are named, and a
      human approved the latest version.
---
