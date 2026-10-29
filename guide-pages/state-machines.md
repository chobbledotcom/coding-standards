---
name: State machines as data
subtitle: Declare the lifecycle once, sweep every cell, derive everything else
guide-category: machines-and-proofs
order: 1
meta_description: >-
  How the tickets repository declares payment lifecycles as executable
  state machines: nodes, events and an exhaustive moves table that is the
  production lookup, swept cell by cell against the real transitions and
  checked for whole-graph laws.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## The problem with lifecycles in code

      A payment row has a state column. Somewhere there are a dozen
      `UPDATE … SET state = ?` statements, a few `if (state === …)`
      branches, a cron job that picks rows in some states, a pruning job
      that deletes rows in others, and a plan document with a table of
      transitions that was accurate on the day it was written.

      Ask "can a row that might hold a customer's money ever be deleted?"
      and nobody can answer without reading all of it. An agent certainly
      cannot, and every change it makes is a chance to break the answer.

      ## A machine is a table the code reads

      The tickets repository declares each lifecycle as data, using a
      shared shape in
      [`src/shared/schema-atlas/machine-spec.ts`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/schema-atlas/machine-spec.ts):

      - **Nodes**: each state, with one or more representative stored rows,
        and facts about the node (can it hold money nobody has accounted
        for? can pruning delete it?).
      - **Events**: each way a row can move, wired to the *real* transition
        function, with facts about the event (does it move money? is it a
        check or a creation?).
      - **A moves table**: for each node, the events that move it and where
        to. **A cell left out is a declared refusal.**

      The worked example is the SumUp recovery machine. SumUp does not sign
      its callbacks and has no redelivery, so one lost callback means a
      paid checkout looks exactly like an abandoned one. The machine
      replaces that lost notice with scheduled checks.

  - type: split-code
    subtitle: The production lookup
    content: |
      ## The moves table

      This is not documentation. It *is* the lookup the production code
      uses to decide where a row goes. `finished` and `unpaid` have no
      moves, so they are terminal. Any event not listed for a node must
      throw.

      From [`sumup-recovery-machine-spec.ts`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/payment/sumup-recovery-machine-spec.ts).
    figure_filename: sumup-recovery-machine-spec.ts
    figure_language: typescript
    figure_code: |
      export const RECOVERY_MOVES: MachineMoves<
        RecoveryNodeId,
        RecoveryEventId
      > = {
        finished: {},
        owed: {
          read_paid_booked: "finished",
          read_paid_contradiction: "owed",
          read_paid_settled: "finished",
          read_paid_unreadable: "owed",
          read_paid_unsettled: "owed",
          read_unavailable: "owed",
        },
        staged: { checkout_created: "waiting" },
        unpaid: {},
        waiting: {
          read_expired_or_failed: "unpaid",
          read_paid_booked: "finished",
          read_paid_contradiction: "owed",
          read_paid_settled: "finished",
          read_paid_unreadable: "waiting",
          read_paid_unsettled: "owed",
          read_pending: "waiting",
          read_unavailable: "waiting",
        },
      };

  - type: markdown
    content: |
      ## Facts on the nodes

      Each node carries the facts the safety rules talk about. Note the
      three-valued answer for money. `waiting` is not "no money owed", it is
      "unknown", and the comment says why:

      ```typescript
      /** Whether a node can be holding money nobody accounted for. `waiting`
       * is unknown rather than no: until SumUp answers, a paid checkout whose
       * callback was lost looks exactly like one nobody ever paid. That is the
       * whole harm this machine exists to close. */
      export type RecoveryOwesMoney = "no" | "unknown" | "yes";

      { id: "waiting", owesMoney: "unknown", prunable: false, ... },
      { id: "owed",    owesMoney: "yes",     prunable: false, ... },
      { id: "unpaid",  owesMoney: "no",      prunable: true,  ... },
      ```

      ## The machine checker: sweep every cell

      A shared test helper,
      [`registerConformanceSweep`](https://github.com/chobbledotcom/tickets/blob/main/test/test-utils/machine-spec.ts),
      runs **every event against every representative row of every node**,
      through the real transition functions:

      > Registers one test per node: every event runs against every one of
      > its shapes and must land where the table says — or throw, when the
      > cell is absent.

      So the table cannot drift from the code. If someone changes a
      transition and not the table, a cell lands somewhere unexpected and
      the sweep fails. If someone adds an event and forgets a node, the
      missing cell is a refusal, and the sweep proves the code really
      refuses. `registerTableChecks` also pins the size of the table (nodes,
      events, shapes), so a quietly shrunk event list cannot pass as "every
      cell still conforms".

      ## Laws over the whole graph

      Beside the sweep sit the laws, written as tests over the declaration,
      not as sentences in a plan. From the machine's test file:

      ```typescript
      test("a row that may hold money is never deleted on age alone", () => {
        expect(unaccountedNodes.map((node) => node.id)).toEqual([
          "waiting",
          "owed",
        ]);
        for (const node of unaccountedNodes) {
          expect(node.prunable, `${node.id} must not be prunable`).toBe(false);
        }
      });

      test("a row that may hold money always has something that will act on it", () => {
        // every unaccounted node has at least one system event that moves it
      });
      ```

      And over the graph, using a shared walker:

      - every node is reachable from a freshly staged row;
      - every node can still reach a closed answer;
      - a closed row has no way back out.

      A law stops a *future* node or event from quietly breaking the rule.
      Add a new state that can hold money and forget to make it
      non-prunable, and the build fails with the law's name.

      ## Derive, never restate

      Once the machine exists, nothing else gets to restate it:

      - The queue of rows worth checking is **derived**: the nodes some
        `check` event can move. A new check event joins the queue by being
        declared.
      - The prunable list is derived from the node facts.
      - The terminal list is derived from the table: a node with no moves.
      - SQL guards, status words, danger flags and the operator's map at
        `/admin/schema` all read from the spec.
      - Vocabulary lists are exhaustive mapped records keyed by the
        machine's union type, so a new or renamed state stops compiling
        until every consumer learns it.

      ## Corrupt stored data is a separate question

      The machine says what valid transitions do. It cannot say what is in
      the database right now. A bounded live scan
      ([`schema-anomaly-scan.ts`](https://github.com/chobbledotcom/tickets/blob/main/src/shared/db/schema-anomaly-scan.ts))
      reads the stored rows, checks the machine's rules against them (for
      example, every state except `staged` must carry a checkout id), and
      shows each fault to the operator. "A proof over valid transitions does
      not detect corrupt stored records."

      ## Contract drift in review

      `deno task contract:diff` prints how this branch changed the declared
      machines: moves, money flags and tables, against the merge base. It
      is a report, not a gate. It goes in the pull request description, so a
      reviewer sees "this branch adds a move from `owed` to `unpaid`" in one
      line instead of finding it in a diff.

  - type: callout
    variant: success
    icon: "mdi:check-circle-outline"
    name: Copy this
    content: |
      Pick your one lifecycle where a bug costs money. Write its states,
      events and moves as a typed object. Make the production code look up
      the next state in that object, and throw when the cell is missing.
      Then write one test that runs every event against every state. That
      alone will find bugs.
---
