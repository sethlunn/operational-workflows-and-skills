# Service Analysis Common

Shared procedure for service or component analysis workflows that combine service understanding, telemetry evidence, and a user-facing artifact such as a Confluence page, health summary, or publishable rollup.

Read `../references/confluence-analysis-writing-standard.md` when writing or updating a Confluence page body.
Read `../references/telemetry-measurability.md` when the question depends on metric dimensions, historical rollups, or distinct-entity conclusions.

## Common Workflow

1. Resolve the target, scope, and output surface.
- Start from the service, component, or code path the user gives.
- Read the local `AGENTS.md` guidance for that codebase and any relevant parent `AGENTS.md`.
- Write down the exact question being answered.
- Distinguish whether the user wants:
  - a quick answer in chat
  - an updated existing Confluence page
  - a new Confluence page
  - a publishable operational summary such as Slack or another destination
- If the user provides an explicit output target, treat it as the canonical output target unless there is a strong reason not to.

2. Set the defaults explicitly.
- Use exact absolute dates in all outputs.
- Default the environment to production unless the user explicitly asks for something else.
- Default to read-only investigation unless publishing is explicitly requested.
- When publishing, decide who the canonical writer is. If there is no larger orchestrator, the current workflow is the canonical writer for its artifact.

3. Build the code-backed understanding first.
- Inspect the local codebase before interpreting telemetry when local code inspection is relevant to the question.
- Use local code to disambiguate service identity, entrypoints, runtime variants, telemetry emitters, or important short-circuit behavior.
- When code inspection matters, record the files that establish:
  - ingress or entrypoint
  - core execution path
  - signal emission
  - important short-circuit or no-op behavior

4. Gather telemetry evidence second.
- Use the narrowest reliable telemetry surface first.
- Prefer topology discovery, exact metric series inspection, and exact filters over broad scans.
- Confirm that the dimensions or tags you intend to filter on are actually retained by the telemetry source.
- Treat fixed windows as the source of truth when long-range scalar rollups disagree with the sum of fixed windows.

5. Separate evidence from interpretation.
- Distinguish:
  - direct evidence from code
  - direct evidence from telemetry
  - interpretation or inference
- Call out what the telemetry can answer directly and what it cannot answer from the available signal.

6. Write the user-facing artifact with caveats in the body.
- Include exact scope, exact dates, and exact filters.
- Put important caveats in the artifact itself, not only in the chat response.
- Include the exact query shapes used for the core evidence when the workflow depends on telemetry analysis.

## Output Rules

- Use exact dates, not relative dates.
- Do not claim distinct-customer or distinct-entity results unless the telemetry preserves identity strongly enough to support that conclusion.
- If an important question cannot be answered from the current telemetry surface, say so explicitly.
- Link the created or updated Confluence page in the final response when page publishing is part of the task.
