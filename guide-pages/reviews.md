---
name: Review rules
subtitle: Answer every thread, fix what is actually broken, and know when to stop
guide-category: workflow
order: 4
meta_description: >-
  The review rules agents follow: answer every thread with a fix and its
  regression test, prefer actually broken over perfect-world findings,
  declare convergence, and never loosen a check under review pressure.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: callout
    variant: info
    icon: "mdi:school-outline"
    name: New to this?
    content: |
      A *review thread* is a comment on a specific line of a pull request
      ([GitHub docs](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/reviewing-changes-in-pull-requests/about-pull-request-reviews)).
      On the tickets repository, every push is reviewed automatically by
      [CodeRabbit](https://www.coderabbit.ai/), an AI review service, and the
      findings are then handled by agents and a person.
  - type: markdown
    content: |
      ## Reviews with agents on both sides

      Every push to the tickets repository starts an automated review, and
      agents address the findings. Without rules, that loop never ends:
      each round produces new suggestions, each suggestion gets a change,
      and each change gets new suggestions. The rules below keep the loop
      short and honest.

      ## Answer every thread you address

      > Reply on each thread with the fix and its regression test, or with
      > why it is not actionable. When a valid suggestion is out of scope,
      > open an issue with enough context, then reply with the link.

      Silently resolving a thread hides whether the finding was fixed,
      rejected or forgotten. A reply with the test name makes the fix
      checkable.

      ## "Actually broken" outranks "nice in a perfect world"

      > A finding earns a fix when it names a concrete failure with real
      > inputs and state. Decline perfect-world work with a short reason, or
      > make it an issue. When consecutive review rounds name no new
      > concrete failure, declare the review converged and hand the decision
      > to a human.

      Review bots, human or automated, can always find something. This rule
      gives the agent a test for which findings matter (real inputs, real
      state, a real failure) and an exit when the findings stop being
      concrete. Converged reviews go to a person.

      ## Whose defect is it?

      - **A branch fixes the defects it introduces.** A finding about code
        the branch adds is that branch's work. Fix it and pin it with a test.
      - **An issue is for a defect already on main.** Do not fix it in the
        current pull request. See
        [stay on your task](/techniques/instructing-agents/guardrails/).
      - **A red check is not automatically yours.** Trace a CI failure far
        enough to name its cause before setting it aside. Say so once, with
        evidence, then re-run. Never ignore a failure. (Unless it looks like
        infrastructure, in which case stop and report.)

      ## Never loosen a check under pressure

      A review finding can sometimes be "closed" by changing the check
      instead of the code: raising a threshold, adding an exemption,
      narrowing a pattern. The rule is absolute:

      > Fix the code, or argue on the thread that the finding is wrong. A
      > deliberate change to a check is an owner decision, never a side
      > effect of review pressure.

      ## Check findings against the real code

      > Check every finding or amendment against the real `src/` and
      > `test/` before you touch the plan. When the built behavior is wrong,
      > fix the code and pin it with a regression test. Amend the contract
      > only when the contract is what is wrong.

      Reviewers, especially automated ones, sometimes describe code that is
      not there, or describe it as it was two commits ago. An agent that
      trusts the finding over the source will "fix" something that was never
      broken.

      ## Stop and redesign

      The pull request workflow has a hard trigger: if review finds two
      cross-cutting state or concurrency bugs, stop patching and go back to
      the [behaviour contract](/techniques/instructing-agents/behaviour-contracts/).
      Two such bugs mean the design is wrong, and a third patch will not fix
      it.
---
