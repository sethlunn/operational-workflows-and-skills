# Diataxis Writing Rules

Use this reference when creating or revising documentation in this repo.

Choose the document type first. Do not start writing until you know whether the reader needs:

- teaching
- task completion
- exact lookup
- understanding

## Quadrant Chooser

- Use a tutorial when the reader is new to the task and should be led through one safe path.
- Use a how-to guide when the reader already understands the domain and wants to accomplish a specific task.
- Use reference when the reader needs exact facts, fields, commands, structures, or contracts.
- Use explanation when the reader needs rationale, tradeoffs, or the mental model behind a design.

If the answer is "more than one", split the content unless the mixed mode is intentional and clearly labeled.

## Tutorial Rules

- Teach through one guided path.
- Optimize for confidence and momentum, not completeness.
- State the starting point and expected end state explicitly.
- Include recovery notes for likely mistakes.
- Keep conceptual digressions short and link out to explanation when needed.

Do not use a tutorial to store every option, edge case, or contract detail.

## How-To Rules

- Start with the task outcome.
- Assume baseline familiarity with the system.
- Give exact steps, inputs, and validation checks.
- Call out failure points and stop conditions.
- Link to reference for exact options or field-level details.

Do not turn a how-to guide into a background essay.

## Reference Rules

- Be exact, stable, and scannable.
- Prefer definitions, lists, schemas, contracts, and field descriptions.
- Organize by lookup shape rather than by narrative flow.
- Include examples only when they clarify the contract.
- Keep subjective recommendations and historical rationale out of the main body.

Do not hide required procedural steps inside reference.

## Explanation Rules

- Answer why the system is shaped this way.
- State tradeoffs, alternatives, and consequences explicitly.
- Prefer coherent mental models over exhaustive step lists.
- Link to tutorials and how-to guides for action.
- Link to reference for exact contracts or commands.

Do not use explanation as a substitute for operational instructions.

## Split Rules

- If a doc mixes task steps and rationale, keep the task in the how-to and move the rationale to explanation.
- If a doc mixes procedure and exact contract details, keep the procedure in the workflow or how-to and move the contract to reference or templates.
- If an output artifact must contain mixed modes, label the sections explicitly so the reader can switch modes on purpose.

## Template Selection

- Use `templates/tutorial-page.md` for guided first-run teaching docs.
- Use `templates/how-to-page.md` for task-oriented docs.
- Use `templates/reference-page.md` for exact lookup docs.
- Use `templates/explanation-page.md` for rationale and mental-model docs.

If a domain-specific template already exists, such as an incident page or child-result contract, prefer that template over the generic one.

Current example tutorial in this repo:

- `tutorials/first-incident-investigation-trial-mode.md`

## Repo-Specific Guidance

- Keep reusable procedure in `workflows/`.
- Keep stable facts and decision rules in `references/`.
- Keep output contracts in `templates/`.
- Keep model-specific behavior thin in `codex/*/SKILL.md`.
- Keep design rationale and architectural context in `explanations/` or review artifacts when it is still evolving.

## Review Checklist

Before merging a documentation change, check:

- Is the document type explicit?
- Does the document stay in one primary mode?
- If it mixes modes, are the sections labeled clearly?
- Is the file in the right repo layer for its job?
- Is there a better existing template for this artifact?

Use `references/diataxis-review-checklist.md` when you want a fuller section-by-section review pass instead of a quick authoring check.
