---
name: Comment limits
subtitle: Comments say only what the code cannot, held to a measured cap
guide-category: static-checks
order: 8
meta_description: >-
  How the tickets repository measured its comments, set a 12-line and
  100-column cap, ratchets it down, and teaches agents to delete narration
  rather than compress it.
blocks:
  - type: guide-header
  - type: guide-navigation
  - type: markdown
    content: |
      ## Agents narrate

      Ask an LLM to write a function and you often get a docstring that
      restates the name, a comment above each block that re-describes the
      block, and a paragraph explaining what the code used to do before
      this change. It reads as diligent, and it is mostly noise. Worse, it
      drifts: the code changes and the paragraph does not.

      AGENTS.md has three short rules about it:

      > **Comments describe current code**: Delete comments that compare
      > current code with an old implementation. Git history keeps the old
      > code.
      >
      > **Comments are short**: A comment only adds what the reader cannot
      > see — a why, a constraint, a surprise. Never re-narrate the lines
      > below in prose, and never restate a name. If a comment grows, fix
      > the tangle it explains.

      ## Measure before you cap

      Before setting a limit, the repository measured its own comments.
      [docs/comment-policy.md](https://github.com/chobbledotcom/tickets/blob/main/docs/comment-policy.md)
      is the result, and it is worth reading as an example of deciding a
      rule from data. A few findings:

      - Comment lines were **18.2%** of `src/`.
      - **16.2%** of all comment churn happened in hunks that changed no
        code at all: pure prose editing.
      - A conservative detector found **454** comments that only restate
        the name below them, like `/** Create */` above `handleCreate`.
      - About **2%** of comments explained what the code *used to* do.
      - The biggest problem was length, not count: "The complaint the data
        actually supports is that comments are too **long**, not that there
        are too many."

      ## The cap

      `deno task check:comments` holds every comment in `src/` to at most
      **12 lines**, and every comment line to at most **100 columns**. The
      numbers live in one place:

      ```typescript
      export const LIMITS: CommentLimits = { maxColumns: 100, maxLines: 12 };
      ```

      They started higher and came down in steps (20, 16, 12), each step
      measured first. The document prices every next step: how many
      comments in how many files would need rewriting to reach 8, 6 or 4.

      A second check, `check:comment-ste`, holds comment prose to the
      mechanical [Simplified Technical English](/techniques/instructing-agents/plain-words/)
      rules.

      ## Delete, do not compress

      The most useful lesson in the document came from the first attempt to
      meet a lower cap:

      > A docstring does not come under a limit by a change of format. The
      > first attempt at 16 lines proved that. Compression to save a line
      > produced run-on sentences and broken grammar. In one file a numbered
      > step list became a 39-word paragraph.

      The next step, from 16 to 12, was done by deletion alone, across 222
      comments in 181 files.

      ## What a comment is for, when an AI reads the code

      The policy is explicit that most readers are now models:

      > An AI writes and reads most of this code. That reader parses the
      > whole file, and it follows a name into another file. So a comment
      > earns its place only when it says something the reader **cannot**
      > get from the code.

      What a reader cannot get from code:

      - a constraint from outside the repository: an edge subrequest
        budget, a provider quirk, a replica catch-up window;
      - a rule the code permits but must never do: "Never gate a security
        decision on this cache";
      - an invariant that lives in another file, and why it holds;
      - a trap, where the obvious reading of the code is wrong.

      And when a long comment is genuinely useful, it is usually in the
      wrong place:

      | The passage is | It belongs in |
      | --- | --- |
      | a business rule | a `specs/` Feature |
      | a design decision, with its options | `docs/` |
      | a deferred idea | a GitHub issue |
      | a per-function contract, in a module header | that function |

  - type: callout
    variant: info
    icon: "mdi:robot-outline"
    name: The judgement half
    content: |
      A cap catches long comments, not pointless short ones. The
      [code grader](/techniques/workflow/code-grader/) asks an LLM the
      judgement question for each file, scored 0 to 3: "Every comment adds
      what the code cannot say, and none re-narrate lines or restate a
      name."
---
