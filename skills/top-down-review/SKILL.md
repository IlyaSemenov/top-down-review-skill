---
name: top-down-review
description: "Code review that starts from the task, not the diff. Reviews a branch, MR/PR, or diff in 3 steps: WHY the change matters and whether its product contract is right, WHAT the solution should be, designed from the task, and HOW the code implements it — correctness, tests, documentation, naming, and prose. Use when the user asks to review changes."
---

# Top-Down Review

Do not edit code unless the user explicitly asked to fix the findings.

## Project review profile

Follow [Project profile](project-profile.md) when the user asks to set up the project review profile, `REVIEW.md`.
During a review, if the project has no profile and the review relied on project-specific checks worth recording, offer one in `To discuss`.

## Process

1. Read [Review method](review-method.md).
2. Determine the review range — the base and the reviewed revision — and use the same range in every later command.
   - For a branch or an MR/PR, the base is the merge base with a freshly fetched target branch, not a stale local ref.
     The target branch is the MR/PR target, otherwise the base branch named in `REVIEW.md`, otherwise the branch `refs/remotes/origin/HEAD` points to.
   - For uncommitted changes, the base is `HEAD`, and the range includes the in-scope untracked files, which `git diff` omits.
   - Keep a range the user defined explicitly.

   If the request defines no range, review the uncommitted changes when there are any, otherwise the current branch.
   If the base or the reviewed revision cannot be determined unambiguously, ask.
3. Review the range following the method, then report the results in the format below.

## Report format

Respond in this order:

- `Summary` — 1-2 lines on the review range and the affected surfaces, and the main problem, if any.
- `Product and architecture assessment` — the business contract, the key invariants, and the target solution the change was assessed against.
- `Findings` — product issues first, then architecture, then implementation; within each layer, by descending severity.
- `Polish` — see [Polish](#polish).
- `Alternatives`.
- `To discuss` — the open questions.
- `What was checked` — what was checked and how, and the residual risk.
  A fully static review is normal; do not run checks just to fill this section.

Omit empty `Polish`, `Alternatives`, and `To discuss`; with no findings, write `Findings: none`.
Every finding and polish fix is meant to be applied as proposed unless `To discuss` questions it.
Do not mention staging, index, or commit status.

### Findings

Number each finding as `F1`, `F2`, … and give it a short descriptive title — what is wrong and where.
Give its severity, file and line, and layer: `product` (WHY), `architecture` (WHAT), or `implementation` (HOW).
When the change already has an earlier review — comments in the MR/PR discussion, a review the user provided, or an earlier round of this review — note a finding the earlier review already raised, and assess the author's replies and fixes by the same criteria as your own findings.
In `To discuss`, refer to a finding by its number and title together, for example `F4 (order.list contract)`.

### Polish

Say in one line which areas were skipped because a finding proposes to rewrite or delete them.
Group polish by file, with one nested line per fix: `line — "old" → "new"`.

## Feedback on a review

When the user or another reviewer points out a missed issue or a wrong finding, or asks to analyze a review, follow [Review feedback](review-feedback.md).
