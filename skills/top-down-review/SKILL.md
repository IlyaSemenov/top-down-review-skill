---
name: top-down-review
description: "Code review that starts from the task, not the diff. Reviews a branch, MR/PR, or diff in 3 steps: WHY the change matters and whether its product contract is right, WHAT the solution should be, designed from the task, and HOW the code implements it — correctness, tests, documentation, naming, and prose. Use when the user asks to review changes."
---

# Top-Down Review

Review a change as a tech lead: first its value and product contract (WHY), then the solution it should have (WHAT), and only then the implementation (HOW).
Code without an obvious bug can still be a finding if it implements a harmful product decision or an objectively worse architecture.
Do not edit code unless the user explicitly asked to fix the findings.

## Project review profile

A project can add review checks and context in `REVIEW.md` at the repository root; apply them in addition to this skill.
Follow [Project profile](project-profile.md) when the user asks to set up a profile.
During a review, if the project has no profile and the review relied on project-specific checks worth recording, offer one in `To discuss`.

## Process

1. Read the project's agent instructions and `REVIEW.md`.
2. Determine the review range — the base and the reviewed revision — and use the same range in every later command.
   - For a branch or an MR/PR, the base is the merge base with a freshly fetched target branch, not a stale local ref.
     The target branch is the MR/PR target, otherwise the base branch named in `REVIEW.md`, otherwise the branch `refs/remotes/origin/HEAD` points to.
   - For uncommitted changes, the base is `HEAD`, and the range includes the in-scope untracked files, which `git diff` omits.
   - Keep a range the user defined explicitly.

   If the request defines no range, review the uncommitted changes when there are any, otherwise the current branch.
   If the base or the reviewed revision cannot be determined unambiguously, ask.
3. Understand the change: read the diff, the changed files at the reviewed revision, the nearest patterns, and existing tests for the same scenario.
   Use the task statement when one exists — the request, the MR/PR description and discussion, the linked issue with its attachments, commit messages — as input, not as a contract.
   Read the documents for the affected area, found through the project's agent instructions, in the base version, because the change may have rewritten them.
4. WHY: assess the product contract, see [Purpose and contract](#purpose-and-contract-why).
5. WHAT: determine the solution the task needs and compare it with the implemented one, see [Target solution](#target-solution-what).
6. HOW: review the implementation, with separate passes over naming and over prose, see [Implementation](#implementation-how).
7. Verify every finding, see [Verifying findings](#verifying-findings).

Scale the depth by risk, not by diff size.
If the change touches authorization, money, data deletion, public contracts, migrations, concurrent operations, trust boundaries, or irreversible actions, also check the callers and consumers of the affected contract, the reasons behind the previous implementation in the history, the effect on existing data, and recovery after a failure.
For a low-risk change, keep WHY and WHAT to 1-2 lines each.

The review is static by default.
To confirm a specific suspected problem, use the shortest reliable check that leaves the working tree unchanged: an existing focused test, typecheck, or a throwaway probe script.
Run broad linters only at the user's direct request.

## What to check

### Purpose and contract (WHY)

- Formulate the product problem, the user, the expected outcome, and the business invariants, and check that the new rules agree with each other, with the domain model, with role permissions, and with neighboring scenarios.
  Reconstruct them from the task statement, the change, the existing behavior, and the tests; the implementation is evidence of the author's intent, not proof that the resulting behavior is right.
- For a bug fix, check that the change removes every symptom and affected surface from the report, on the report's data when it has some.
- A change that extends or reinterprets the task statement is a finding only when the product decision is harmful or contradicts an explicit requirement.
  A mockup detail may be an example rather than a requirement.
- Look for footguns — behavior that provokes a mistake, irreversible loss, a hidden data change, or a bypass of the expected process — and assess the effects on other roles, existing data, and integrations.
- If the business meaning is ambiguous, name the assumption in `To discuss` and continue the assessment under it.

### Target solution (WHAT)

- Determine the simplest solution that fully covers the requirements within the project's architecture: the owner of each rule, the boundaries, the minimal contracts, and the source of truth.
  Do not build in speculative flexibility.
- Do not treat the abstractions, responsibilities, or data flow chosen in the change as requirements: judge them by what the task needs, not by the fact that they already exist.
- An alternative is a finding only when you explored it enough to propose it with confidence and its advantage — less coupling or fewer entities, an explicit invariant, a restored dependency direction, simpler extension, or a lower real risk of errors — is material and worth the cost and risk of the rework.
  Otherwise put it under `Alternatives`, or under `To discuss` when the tradeoff needs a decision; if the implemented option is no worse, accept it.
- Report anything the change adds that is not needed as a finding, not under `Alternatives`:
  - a second source of truth for the same rule or value;
  - state or a handler that an existing owner already covers;
  - a layer that can be removed.
- Report runtime code as a finding when a declarative mechanism the project uses, such as CSS, a schema constraint, or a type, fully expresses the same behavior.
- Do not demand rewriting neighboring code unrelated to the change.

### Implementation (HOW)

#### Correctness

- Check whether a new data shape exposes data of deleted, hidden, anonymous, or inaccessible entities.
- Check that the UI shows actions to the same users the backend allows to perform them.
- If the change allocates a value that must be unique, such as `max()+1` or a counter, check that concurrent operations cannot both commit the same value.
- When the change introduces a constant — a limit, a threshold, a period — search for an existing setting or constant governing the same quantity and check that they agree.
- If the change alters a schema, an API, a storage format, configuration, or dependencies, check old and new consumers, including mixed versions during a rollout, and rollback, rerun, and partial failure of a migration.
  For a dependency upgrade, check the changelog of the versions actually used.
- If the change replaces an implementation of existing behavior, such as a conversion, a parser, or a query, compare the old and new results on the same inputs, especially on inputs one side cannot represent.

#### Contracts and boundaries

- For each new entry point, file, and public export, check that its module and layer match the owner of its contract — domain API, transport schema, response projection, persistence, UI, or test helper — according to the project's boundary documentation.
- Check whether new public types can express an invalid state or add unjustified optional fields.
- For a new public export, check that its declaration states the contract callers rely on; an inferred type is fine when the inferred shape is the intended contract.
- After a move or refactor, no old paths, wrappers, dead branches, or compatibility TODOs may remain unless existing consumers still need them.

#### Naming

- For each new name of a domain entity or concept — in identifiers, comments, user-facing texts, and documentation — search with `rg` for how it is already named in the schema, types, API, documentation, and UI.
  Local names of variables and helpers need no search.
  A new name for an already named entity is `major` when it enters a public or persisted contract — an API, a URL, a database schema, an event — or spreads across modules; otherwise it is `minor` or `Polish`.

#### Prose

- User-facing texts must read as natural product wording, rather than code identifiers or literal translations.
- Flag filler in texts, documents, and comments: text that states the obvious, retells the code, repeats nearby text, or hedges generically.

#### Tests

- For each new or changed behavior contract, name the invariant it must keep and the realistic breakage a test should catch: an inverted access condition, removed error handling, a changed order of operations, or a shifted boundary value.
  For a bug fix or a behavior change, one such breakage is a return to the old behavior where the new contract requires a different outcome.
  If the tests would still pass after that breakage, they check an indirect effect or the shape of the implementation, and the gap is a finding unless you state why the risk is not real.
  Answer this mentally; do not modify files or run mutation tests.
  If the answer depends on behavior the code does not define, such as row order without `ORDER BY`, treat it as an assumption.
- Propose a test only when its expected result can be derived independently of the code under test; when a test could only restate the code, such as the branches of a template or the values its own mock returns, there is no gap.
  A test in the change that restates the implementation is a finding; the fix may be to delete it.
- The fixtures must not already satisfy the expected outcome: an ordering assertion over rows inserted in the expected order, or a filter over data where every row matches, proves nothing.
- Check the test level against the project's testing conventions.

#### Documentation

- If the change alters product behavior, a workflow, a boundary, or a development rule, check the owning document, found through the project's agent instructions rather than by the nearest changed file.
  If users see the change, also check the user-facing documentation.
- Normative documentation describes the current state, not its history, and does not duplicate a rule owned elsewhere.

## Verifying findings

Keep a finding only if you can answer each question:

- Which concrete scenario or consumer exposes the problem?
- What confirms it and every assumption behind it: code, a contract, a test, or documentation?
  For the behavior of a framework, ORM, database, or library, check the version the project actually uses, from the lockfile, even when it is not installed locally.
- Does the change introduce or materially worsen the problem, or leave unsolved what its task requires?
  Mention a material problem that fails this check in one line as outside the change.
- Which minimal change fixes its root cause, and does it work in the proposed form?
  When the finding prescribes a specific shape of the fix, sketch it against the actual code.

Drop a finding that fails the check; do not downgrade an unproven hypothesis to `minor` or to a question.
An open product decision stays in `To discuss` under an explicit assumption, see [Purpose and contract](#purpose-and-contract-why).
If confirmation needs a runtime check you did not run, report a residual risk and name the check.

## Report format

Respond in this order:

- `Summary` — 1-2 lines on the review range and the affected surfaces, and the main problem, if any.
- `Product and architecture assessment` — the business contract, the key invariants, and the target solution the change was assessed against.
- `Findings` — product issues first, then architecture, then implementation; within each layer, by descending severity.
- `Polish` — see [Polish](#polish).
- `Alternatives` — useful options equivalent to the implemented one or not worth the switch; not micro-style.
- `To discuss` — open questions, assumptions the conclusion rests on, and decisions that need confirmation; only what affects the conclusion or further actions.
- `What was checked` — what was checked and how, and the residual risk: what remained unchecked or uncertain.
  A fully static review is normal; do not run checks just to fill this section.

Omit empty `Polish`, `Alternatives`, and `To discuss`; with no findings, write `Findings: none`.
Every finding and polish fix is meant to be applied as proposed unless `To discuss` questions it.
Do not mention staging, index, or commit status.

### Severity

Severity is the cost of leaving a finding unfixed.

- `blocker` — the fix cannot be deferred: behavior, access or security, data, or a public contract breaks.
- `major` — significant technical debt, an architectural mistake, a fragile contract, or an uncovered risk with a real cost.
  Deferring the fix is acceptable only as explicitly accepted technical debt; propose it in `To discuss` and state what the debt costs.
- `minor` — a local improvement of readability, naming, a test, or documentation; explain its harm.

### Findings

Number each finding as `F1`, `F2`, … and give it a short descriptive title — what is wrong and where.
Give its severity, file and line, and layer: `product` (WHY), `architecture` (WHAT), or `implementation` (HOW).
Write the concrete harm and propose a concrete change, not general advice.
When a finding proposes reworking the solution, name the tradeoff that justifies the rework.
Do not leave withdrawn findings or severity changes made while reasoning in the list.
When the change already has an earlier review — comments in the MR/PR discussion, a review the user provided, or an earlier round of this review — note a finding the earlier review already raised, and assess the author's replies and fixes by the same criteria as your own findings.
In `To discuss`, refer to a finding by its number and title together, for example `F4 (order.list contract)`.

### Polish

Polish is a local fix that changes neither behavior nor a contract anything relies on and whose replacement explains itself: wording, minor naming, phrasing of comments and docs, formatting, small consistency fixes.
Removing a parameter, an export, or a branch without consumers is polish; when a consumer may exist outside the reviewed code, such as a public API, it is a finding.
If the harm needs explaining, it is a finding.

List all polish you found, even next to blocker or major findings, except mechanical formatting, such as indentation, spacing, or line wrapping, when the project configures a formatter or a linter.
Skip polish only in code that a finding proposes to rewrite or delete, and say in one line which areas were skipped.
Group polish by file, with one nested line per fix: `line — "old" → "new"`; merge fixes of the same kind into one line.

## Feedback on a review

When the user or another reviewer points out a missed issue or a wrong finding, or asks to analyze a review, follow [Review feedback](review-feedback.md).
