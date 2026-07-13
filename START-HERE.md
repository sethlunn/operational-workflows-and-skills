# Start Here

This repo uses the Diataxis model:

- tutorials teach through a guided first run
- how-to workflows help complete an operational task
- references and templates provide exact facts and output contracts
- explanations provide rationale and mental models

Codex and Claude Code use thin model-specific adapters over the same shared workflows. Choose the section that matches your need now.

## If You Need A Managed Agent Session

- Start Codex with current skill links and update handling: [scripts/codex-session](./scripts/codex-session)
- Start Claude Code with current skill links and native update handling: [scripts/claude-session](./scripts/claude-session)
- Install links without starting an agent: [scripts/link-codex-skills](./scripts/link-codex-skills) or [scripts/link-claude-skills](./scripts/link-claude-skills)
- Start a preflighted incident session: [scripts/codex-incident-session](./scripts/codex-incident-session) or [scripts/claude-incident-session](./scripts/claude-incident-session)

Read the setup and source-selection details in [README.md](./README.md#using-this-repo-with-codex) and [README.md](./README.md#using-this-repo-with-claude-code).

## If You Need A Guided First Run

Start with [tutorials/first-incident-investigation-trial-mode.md](./tutorials/first-incident-investigation-trial-mode.md) for a safe PagerDuty incident investigation that does not publish.

## If You Need To Do Work Now

Go to `workflows/`. Use the model-native adapter inventory in [README.md](./README.md#current-entry-skills) when you want the corresponding `$skill-name`.

Incident and telemetry work:

- PagerDuty incident to evidence-backed analysis or Confluence write-up: [workflows/pagerduty-incident-analysis.md](./workflows/pagerduty-incident-analysis.md)
- Dynatrace rollout, incident, service-debugging, or exact-id router: [workflows/dynatrace-investigation.md](./workflows/dynatrace-investigation.md)
- PagerDuty-owned service health rollup: [workflows/pagerduty-assigned-service-health.md](./workflows/pagerduty-assigned-service-health.md)
- Incident conclusion validation and Jira follow-up planning: [workflows/incident-followup-planning.md](./workflows/incident-followup-planning.md)

Service analysis and documentation:

- Diataxis-aligned service documentation set: [workflows/service-system-documentation.md](./workflows/service-system-documentation.md)
- Endpoint inventory and production traffic analysis: [workflows/service-endpoint-traffic-analysis.md](./workflows/service-endpoint-traffic-analysis.md)
- Emitted-metric and telemetry analysis: [workflows/service-metric-analysis.md](./workflows/service-metric-analysis.md)

Engineering design and delivery:

- Planned-change technical design: [workflows/technical-design-documentation.md](./workflows/technical-design-documentation.md)
- Bounded code-writing delegation to APEX for a draft PR: [workflows/apex-agent-delegation.md](./workflows/apex-agent-delegation.md)
- Focused Mermaid PR description: [workflows/pr-diagram.md](./workflows/pr-diagram.md)

Review, coaching, and feedback:

- PR-author coaching from recent history: [workflows/pr-author-coaching.md](./workflows/pr-author-coaching.md)
- Author-side PR review triage and fixes: [workflows/babysit-pr.md](./workflows/babysit-pr.md)
- Independent requirement-driven PR review: [workflows/review-pr.md](./workflows/review-pr.md)
- FY26 performance peer-review drafts: [workflows/peer-review.md](./workflows/peer-review.md) (Claude adapter only)

## If You Need Exact Facts

Go to `references/`, `templates/`, or `scripts/`.

Authoring and routing:

- Diataxis authoring rules: [references/diataxis-writing-rules.md](./references/diataxis-writing-rules.md)
- Diataxis review checklist: [references/diataxis-review-checklist.md](./references/diataxis-review-checklist.md)
- Technical-design writing rules: [references/technical-design-writing-rules.md](./references/technical-design-writing-rules.md)
- Confluence routing: [references/confluence-routing.md](./references/confluence-routing.md)
- Git/Jira branch naming: [references/git-branch-naming.md](./references/git-branch-naming.md)

Investigation and review evidence:

- Dynatrace query shapes: [references/dynatrace-query-patterns.md](./references/dynatrace-query-patterns.md)
- Fast-path telemetry triage: [references/dynatrace-fast-path.md](./references/dynatrace-fast-path.md)
- PR coaching rubric: [references/pr-coaching-rubric.md](./references/pr-coaching-rubric.md)
- Peer-review rubric: [references/peer-review-rubric.md](./references/peer-review-rubric.md)
- Subagent usage rules: [references/subagent-usage.md](./references/subagent-usage.md)

Output contracts:

- Parent incident page: [templates/incident-analysis-page.md](./templates/incident-analysis-page.md)
- Incident child investigation: [templates/dynatrace-investigation-result.md](./templates/dynatrace-investigation-result.md)
- Non-incident child analysis: [templates/analysis-child-result.md](./templates/analysis-child-result.md)
- Technical design: [templates/technical-design-document.md](./templates/technical-design-document.md)
- APEX delegation handoff: [templates/apex-agent-handoff.md](./templates/apex-agent-handoff.md)
- Peer-review entry: [templates/peer-review-entry.md](./templates/peer-review-entry.md)
- Generic Diataxis templates: [tutorial](./templates/tutorial-page.md), [how-to](./templates/how-to-page.md), [reference](./templates/reference-page.md), and [explanation](./templates/explanation-page.md)

## If You Need To Understand The System

Start with [explanations/repo-architecture.md](./explanations/repo-architecture.md).

Then choose the explanation that matches the question:

- Service documentation sets: [explanations/service-documentation-pattern.md](./explanations/service-documentation-pattern.md)
- How the incident skill family fits together: [explanations/incident-analysis-family.md](./explanations/incident-analysis-family.md)
- Parent-and-child incident investigation: [explanations/incident-analysis-pattern.md](./explanations/incident-analysis-pattern.md)
- Dynatrace router and branch model: [explanations/dynatrace-investigation-pattern.md](./explanations/dynatrace-investigation-pattern.md)
- Ambiguous telemetry interpretation: [explanations/dynatrace-evidence-interpretation.md](./explanations/dynatrace-evidence-interpretation.md)

These explain why operating logic stays in shared Markdown, why Codex and Claude adapters stay thin, why parent workflows remain canonical writers, and why references and templates remain separate from procedures.

## If You Want Concrete Examples

- Browse the published Compliance documentation source set: [reviews/service-docs/decisioning-services-compliance/](./reviews/service-docs/decisioning-services-compliance/)
- Browse the published Machine Learning Gateway documentation source set: [reviews/service-docs/provider-gateways-machine-learning-gateway/](./reviews/service-docs/provider-gateways-machine-learning-gateway/)
- Browse design assessments and the architecture proposal: [reviews/design/](./reviews/design/)

The service-documentation directories retain Markdown and Mermaid sources, rendered diagrams, and Confluence manifests with stable publication ids.

## If You Are New To The Repo

Use this order:

1. Read the architecture and capability summary in [README.md](./README.md).
2. Read [explanations/repo-architecture.md](./explanations/repo-architecture.md).
3. Install the Codex or Claude skill links, or use the corresponding managed session launcher.
4. Pick one workflow from the task groups above.
5. Open supporting references and templates only when that workflow calls for them.

## Current Documentation Gap

The repo is strongest in how-to guides, reference, and real service-documentation examples. The tutorial quadrant still has only one guided first run.

The next high-value tutorials are:

- adding a shared workflow plus both thin model adapters
- completing a first publish-mode incident write-up
- producing a service documentation set from code discovery through rendered diagrams and publication manifest
