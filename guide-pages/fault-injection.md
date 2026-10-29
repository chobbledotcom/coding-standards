---
name: Fault injection and crash windows
subtitle: Real failures at the database boundary, for every await between two writes
guide-category: testing
order: 6
meta_description: >-
  How the tickets repository tests what happens when a write fails half
  way through a flow: crash windows, SQLite triggers as real faults,
  observers that record violations, and concurrency tests.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      [Fault injection](/glossary/#fault-injection) means breaking something on
      purpose in a test. Here the break is an
      [SQLite trigger](https://www.sqlite.org/lang_createtrigger.html): a small
      rule stored in the database that runs before a write and can abort it with
      `RAISE`. A *durable write* is one saved to the database, which survives a
      restart. A *transaction* groups writes so they all happen or none do
      ([SQLite docs](https://www.sqlite.org/lang_transaction.html)).
  - type: markdown
    content: |
      ## The bug in the gap

      The most expensive payment bugs live in the gap between two durable
      writes. The provider takes the money, then the local write fails.
      The refund goes out, then the server restarts before recording it.
      Happy-path tests never see these, and agents rarely think to write
      them.

      The design guide's rule is to enumerate them:

      > **Enumerate flows × crash windows.** List every flow; every await
      > between two durable writes is a window; each window gets an
      > idempotent-redelivery proof or a fault-injected test.

      ## Faults at the real boundary, not stubs

      Stubbing a database call to throw tests the stub. It does not show
      that the transaction rolls back, that the partial state is what you
      think, or that the retry finds the work. So faults in tickets are
      SQLite triggers, installed for the length of one test:

      ```typescript
      /**
       * Run `body` with one database fault installed — a trigger that makes
       * a chosen write fail at SQLite's own boundary — then lift the fault.
       * Real faults at the storage layer, not stubs: the failing statement
       * rolls back exactly as a production write would.
       */
      export const withDbFault = async <T>(
        createTrigger: string,
        name: string,
        body: () => Promise<T>,
      ): Promise<T> => {
        await execute(createTrigger);
        try {
          return await body();
        } finally {
          await execute(`DROP TRIGGER IF EXISTS ${name}`);
        }
      };
      ```

      A named fault then says exactly which write refuses:

      ```sql
      CREATE TRIGGER test_authority_retirement_fault
        BEFORE UPDATE ON payment_charges
        WHEN NEW.refund_local_state = 'recorded'
        BEGIN
          SELECT RAISE(ABORT, 'authority retirement unavailable');
        END
      ```

      From
      [`test/test-utils/db-fault.ts`](https://github.com/chobbledotcom/tickets/blob/main/test/test-utils/db-fault.ts).
      The same idea runs in the nightly sandbox run: a [Stripe](https://docs.stripe.com/testing) refund
      succeeds at Stripe while a persistent trigger makes the local money
      write fail, and the scenario checks that the booking stays protected,
      a stale form cannot refund twice, and a refresh completes the record
      once the fault lifts.

      ## Observers that cannot be fooled later

      Some invariants are about *every* write, not the final state. A claim
      on a payment row must belong to the right worker at the moment it is
      armed, even if a later write tidies up. The payment seam observer
      ([`test/test-utils/payment-seam.ts`](https://github.com/chobbledotcom/tickets/blob/main/test/test-utils/payment-seam.ts))
      records claim and refund-arm facts through test-only triggers as they
      happen:

      > A later settlement cannot erase a recorded violation. The observer
      > checks the authority revision and request identity at arm time.

      ## Concurrency, directly

      The behaviour contract has a concurrency table, and each row becomes
      a test that runs the two operations together: a stale form against a
      newer save, a scheduled retry against a callback replay, two
      reservations for the last place. The production suite runs a
      concurrent-reservation race every night, and AGENTS.md lists the
      metamorphic properties that must hold: replay does not double-credit,
      capacity never goes below zero.

      ## Where the proofs take over

      Fault tests cover the windows someone listed. The
      [state machines](/techniques/machines-and-proofs/state-machines/) and
      [Apalache models](/techniques/machines-and-proofs/tla-and-apalache/)
      cover the rest: for example, that two workers on one payment row can
      never release or overwrite each other's claim, for any interleaving.
---
