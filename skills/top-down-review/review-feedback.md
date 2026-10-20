# Review feedback

Verify the feedback independently.
If it confirms an error or an omission, including a wrong severity or fix, correct the report and find which step or criterion of the review process caused it.
Otherwise, explain why the assessment stands.

Turn a cause into a rule only if the rule:

- concerns the review process or the project's review checks, not the implementation of a specific feature
- will be useful in future reviews of different areas
- does not duplicate the project's agent instructions or documentation
- is phrased as a short general principle without a historical case

Route each rule by where it belongs:

- A review process rule useful for any project goes to this skill's source repository, not to the installed copy, which an update replaces.
  Strike file, feature, and technology names from it; if no rule remains without them, it belongs to `REVIEW.md`.
- A review check specific to the project goes to `REVIEW.md`.
- A development rule goes to its owning project document.

Propose the rules to the user and apply them only after the user agrees.
