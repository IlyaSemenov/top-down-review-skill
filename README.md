# Top-Down Review

A code review skill that starts from the task, not the diff.

It reviews a branch, MR/PR, or diff in 3 steps:

1. **WHY** — what the change is for, which product invariants and user scenarios it must preserve, and whether the requirement itself is right.
2. **WHAT** — the minimal sufficient solution within the existing architecture, which the reviewer designs from the task and compares with the implemented one.
3. **HOW** — correctness, contracts, complexity, tests, naming, documentation, and prose.

It runs as a single pass by one agent.
The report groups findings by layer — WHY, WHAT, HOW — ranks them by severity within each layer, and lists self-explanatory fixes — wording, minor naming, formatting — in a compact `Polish` section, so they are not lost next to major findings.

## Install

Install the skill into the project and commit the installed copy, so every developer and cloud agent gets the same version:

```bash
npx skills add https://github.com/IlyaSemenov/top-down-review-skill --skill top-down-review
```

The command writes the skill to `.agents/skills/top-down-review/` and records its source in `skills-lock.json`.

Update it with:

```bash
npx skills update top-down-review -p
```

The update replaces the installed copy entirely.
Do not edit the copy: propose general changes to this repository and keep project-specific checks in the project review profile.

## Project review profile

A project complements the skill with a review profile: `REVIEW.md` at the repository root with review checks that the skill and the general project documentation do not cover, such as tenant isolation or the authorization required for new entry points.
Claude Code Review reads the same file, so one profile serves both.

Ask your agent to set up the profile, for example in Claude Code:

```text
/top-down-review set up the review profile for this project
```

## Usage

Invoke the skill by name — `/top-down-review` in Claude Code, `$top-down-review` in Codex, `/skill:top-down-review` in pi — or ask your agent to review the changes.
For example, in Claude Code:

```text
/top-down-review the current branch
```
