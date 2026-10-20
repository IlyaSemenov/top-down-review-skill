# Project profile

Set up or update the project review profile, `REVIEW.md` at the repository root, when the user asks for it.

## Collect the checks

1. Read the project's agent instructions, its documentation index, and the existing `REVIEW.md`, if any.
2. Find other review guidance in the project, such as review checklists, PR templates, contributing guides, or earlier review prompts and skills.
3. Collect review checks that a reviewer must apply in this project and that neither the skill's general criteria nor the project documentation already state.
   Typical checks name domain-specific risk areas, such as tenant isolation or the authorization required for a new entry point, and point to the documents that define them.
   Under `Task statement`, record where tasks and MRs live, how branches, commits, and MR titles refer to them, and which tool reads them, if any.
4. Leave out development rules: when a rule belongs to the project's conventions, propose adding it to its owning document instead, so the reviewer finds it through the project's documentation.

## Write the profile

Write the profile in the project's documentation language.
Other review tools, such as Claude Code Review, read the same file, so update an existing `REVIEW.md` instead of replacing it.

Keep the checks to a short list with links to the owning documents.
Link to the review guidance found in the project instead of restating it, and report where that guidance duplicates this skill or contradicts the profile.

```markdown
# Code review

Review context and checks specific to this project.

## Task statement

- Branches `feature/<N>` and MR titles `<N>: …` refer to issue `PROJ-<N>` in Jira.
- Read the issue with `jira issue view PROJ-<N> --comments`.
- MRs live in GitLab: read the MR description and discussion with `glab mr view <branch> --comments`.

## Checks

- A new API procedure requires authorization and checks the permission of the acting role — see [Access](docs/product/access.md).
- A new HTTP handler for product data sets the tenant context before reading the session or querying the database — see [Web app](docs/architecture/web-app.md).
```

Name the base branch in the profile only if `refs/remotes/origin/HEAD` does not point to it.
