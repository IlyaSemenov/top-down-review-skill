# Top-Down Review Development

You are developing the `top-down-review` skill in `skills/top-down-review/`, not running a review.

Use these role names consistently while editing the skill:

- **Developer agent** — the agent modifying this repository.
- **Reviewer** — the agent executing the installed skill for a user.

Keep developer instructions in this file.
Keep runtime instructions for the reviewer in `skills/top-down-review/SKILL.md` and its linked runtime documents.

## Documentation rules

- Keep the skill project-agnostic: project-specific checks belong in the project review profile described in `skills/top-down-review/project-profile.md`.
- Refer to project conventions through the project's agent instructions and documentation, never through concrete project file paths.
- Write the skill in English.
- Write Markdown in One Sentence Per Line format and separate meaningful paragraphs with a blank line.
