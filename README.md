# Operational Workflows and Skills

Portable, company-specific operational workflows and reusable Codex and Claude Code skill adapters for LLM agents.

This repo keeps the reusable operating model in shared Markdown and keeps model-specific skill wrappers thin. The goal is to make one investigation or analysis pattern reusable across Codex and Claude Code today, without duplicating the procedure in each adapter.

Start with [START-HERE.md](./START-HERE.md).

## Use This Repo By Need

Diataxis is the simplest useful lens for this repo: choose documentation based on what the reader needs right now.

### Tutorials

Use tutorials when the goal is to learn safely through a guided first run.

Current tutorial:

- [tutorials/first-incident-investigation-trial-mode.md](./tutorials/first-incident-investigation-trial-mode.md)

### How-to Guides

Use how-to guides when the goal is to complete an operational task.

The main how-to surface is `workflows/`.

Good starting points:

- [workflows/pagerduty-incident-analysis.md](./workflows/pagerduty-incident-analysis.md)
- [workflows/dynatrace-investigation.md](./workflows/dynatrace-investigation.md)
- [workflows/technical-design-documentation.md](./workflows/technical-design-documentation.md)
- [workflows/implement-jira-story.md](./workflows/implement-jira-story.md)
- [workflows/service-system-documentation.md](./workflows/service-system-documentation.md)
- [workflows/pagerduty-assigned-service-health.md](./workflows/pagerduty-assigned-service-health.md)
- [workflows/service-endpoint-traffic-analysis.md](./workflows/service-endpoint-traffic-analysis.md)
- [workflows/service-metric-analysis.md](./workflows/service-metric-analysis.md)
- [workflows/incident-followup-planning.md](./workflows/incident-followup-planning.md)
- [workflows/apex-agent-delegation.md](./workflows/apex-agent-delegation.md)
- [workflows/pr-author-coaching.md](./workflows/pr-author-coaching.md)
- [workflows/babysit-pr.md](./workflows/babysit-pr.md)
- [workflows/review-pr.md](./workflows/review-pr.md)
- [workflows/pr-diagram.md](./workflows/pr-diagram.md)
- [workflows/peer-review.md](./workflows/peer-review.md)

### Reference

Use reference when you need exact facts, patterns, or output contracts.

The main reference surface is:

- [references/](./references/)
- [templates/](./templates/)
- [scripts/](./scripts/)
- [mermaid.config.json](./mermaid.config.json)

High-value reference files:

- [references/diataxis-writing-rules.md](./references/diataxis-writing-rules.md)
- [references/diataxis-review-checklist.md](./references/diataxis-review-checklist.md)
- [references/incident-analysis-surfaces.md](./references/incident-analysis-surfaces.md)
- [references/confluence-routing.md](./references/confluence-routing.md)
- [references/dynatrace-query-patterns.md](./references/dynatrace-query-patterns.md)
- [references/dynatrace-fast-path.md](./references/dynatrace-fast-path.md)
- [references/technical-design-writing-rules.md](./references/technical-design-writing-rules.md)
- [references/git-branch-naming.md](./references/git-branch-naming.md)
- [references/pr-coaching-rubric.md](./references/pr-coaching-rubric.md)
- [references/peer-review-rubric.md](./references/peer-review-rubric.md)
- [references/subagent-usage.md](./references/subagent-usage.md)
- [templates/tutorial-page.md](./templates/tutorial-page.md)
- [templates/how-to-page.md](./templates/how-to-page.md)
- [templates/reference-page.md](./templates/reference-page.md)
- [templates/explanation-page.md](./templates/explanation-page.md)
- [templates/incident-analysis-page.md](./templates/incident-analysis-page.md)
- [templates/dynatrace-investigation-result.md](./templates/dynatrace-investigation-result.md)
- [templates/analysis-child-result.md](./templates/analysis-child-result.md)
- [templates/technical-design-document.md](./templates/technical-design-document.md)
- [templates/apex-agent-handoff.md](./templates/apex-agent-handoff.md)
- [templates/peer-review-entry.md](./templates/peer-review-entry.md)
- [scripts/render-mermaid-diagrams](./scripts/render-mermaid-diagrams)
- [mermaid.config.json](./mermaid.config.json)

### Explanation

Use explanation when you need rationale, tradeoffs, or the mental model behind the workflows.

Start with:

- [explanations/repo-architecture.md](./explanations/repo-architecture.md)
- [explanations/service-documentation-pattern.md](./explanations/service-documentation-pattern.md)
- [explanations/incident-analysis-family.md](./explanations/incident-analysis-family.md)
- [explanations/incident-analysis-pattern.md](./explanations/incident-analysis-pattern.md)
- [explanations/dynatrace-investigation-pattern.md](./explanations/dynatrace-investigation-pattern.md)
- [explanations/dynatrace-evidence-interpretation.md](./explanations/dynatrace-evidence-interpretation.md)
- [reviews/design/skills-architecture-and-governance.md](./reviews/design/skills-architecture-and-governance.md)

Some explanation-oriented material still lives under `references/` and should move over time:

- [references/incident-investigation-lessons-2026-03-27.md](./references/incident-investigation-lessons-2026-03-27.md)

## Core Model

This repo is organized around a small set of layers:

1. `codex/` and `claude/`
   Thin, model-specific entry adapters that tell an agent when to trigger and which shared workflow to read.
2. `workflows`
   The real reusable procedure.
3. `references`
   Stable facts, query patterns, routing rules, and caveat handling.
4. `templates`
   Output structure for Confluence pages and other reusable artifacts.
5. `tutorials` and `explanations`
   Guided learning material and the rationale behind the operating model.
6. `reviews`
   Design, code, and planning assessments.
7. `scripts`
   Helper launchers and setup utilities.

The operating rule is simple:

```text
user request
  -> entry skill
  -> shared workflow
  -> optional branch workflow
  -> references/templates/scripts as needed
  -> final artifact
```

## Design Principles

- Keep both Codex and Claude `SKILL.md` files small and routing-oriented.
- Put reusable business logic in `workflows/`, not in skill wrappers.
- Put stable supporting knowledge in `references/`, not in workflows unless it is procedural.
- Put output shape in `templates/`, not process.
- Add scripts only when a step is brittle or benefits from deterministic execution.
- Prefer exact dates, exact ids, and exact query shapes over vague summaries.
- Separate direct evidence from interpretation in every analysis workflow.
- Use one canonical writer for any published parent artifact.
- Keep Mermaid source in git and render checked-in SVG snapshots for reader-facing docs.

## Diagram Snapshotting

For service-doc diagrams, prefer:

```text
reviews/service-docs/<service-slug>/
  diagrams/
    <diagram-name>.mmd
    rendered/
      <diagram-name>.svg
```

Rules:

- Keep Mermaid source in `.mmd` files.
- Keep rendered `.svg` snapshots checked in beside the docs.
- Refresh snapshots with `./scripts/render-mermaid-diagrams`.
- Use the shared `mermaid.config.json` so snapshots stay visually consistent across services.

## Skill Families

The repo is easier to reason about as a small set of families rather than as a flat list of unrelated skills.

### Incident And Operational Orchestration

- `pagerduty-incident-analysis`
  End-to-end incident orchestrator from PagerDuty through Dynatrace to Confluence.
- `incident-followup-planning`
  Validate incident conclusions and create actionable Jira follow-up stories.

These start from an incident or incident-derived artifact and usually produce or update a canonical parent document.

### Telemetry Investigation Workers

- `dynatrace-investigation`
  Shared Dynatrace investigation router for rollout checks, incident debugging, service debugging, and GUID tracing.

This can run standalone or as a bounded child investigation inside a larger workflow.

### Service Analysis

- `service-system-documentation`
  Produce a Diataxis-aligned documentation set for a service or larger system using code-backed discovery and Dynatrace telemetry.
- `service-endpoint-traffic-analysis`
  Inspect code, inventory endpoints, map them to Dynatrace traffic, and publish a cleanup-oriented Confluence page.
- `service-metric-analysis`
  Inspect code, identify emitted metrics, analyze telemetry history and breakdowns, and publish a Confluence page.
- `pagerduty-assigned-service-health`
  Discover the current user's PagerDuty-owned services, map them to Dynatrace production entities, and summarize health over an exact time window.

These now share a common service-analysis layer:

- [workflows/service-analysis-common.md](./workflows/service-analysis-common.md)
- [references/telemetry-measurability.md](./references/telemetry-measurability.md)
- [references/confluence-analysis-writing-standard.md](./references/confluence-analysis-writing-standard.md)

### Engineering Design And Delivery

- `technical-design-documentation`
  Draft or revise a future-state technical design from Jira or PRD requirements, current code, solution options, and rollout constraints.
- `implement-jira-story`
  Read a complete Jira story and its comments, prepare a safe branch or worktree from the latest master, implement the code locally, and verify it against the effective contract.
- `apex-agent-delegation`
  Package bounded code-writing work for an APEX-launched agent that opens a draft PR, while keeping review, documentation, runtime verification, and finishing work local.
- `pr-diagram`
  Write or revise a PR description around one focused Mermaid flow or sequence diagram.

These capabilities cover the path from design through local Jira-story implementation or bounded implementation delegation and reviewer-facing change communication. APEX is intentionally limited to code-writing draft PRs; it is not a runtime for incident, telemetry, documentation-publishing, or PR-review workflows.

### Review, Coaching, And Feedback

- `pr-author-coaching`
  Coaching workflow that analyzes one or more engineers by recent PR history, review themes, and code quality patterns.
- `babysit-pr`
  Active code-owner workflow for triaging review comments, making safe revisions, pushing updates, and replying to review threads.
- `review-pr`
  Requirement-driven PR review workflow that reads Jira, epic, and Confluence context before reviewing the code and discussing findings in session.
- `peer-review`
  Claude-only FY26 performance-feedback workflow that combines delivered Jira work, GitHub review signals, and user-provided observations into evidence-backed draft Lattice entries.

These are intentionally separate:

- `pr-author-coaching`
  Use when the starting object is one or more GitHub usernames and the goal is recurring coaching guidance across recent PRs.
- `babysit-pr`
  Use when you own the PR and want the agent to move it forward by handling review feedback.
- `review-pr`
  Use when you want an independent review of the PR against the story, design context, and current code.
- `peer-review`
  Use when the subject is a person's performance feedback, not the correctness of a pull request.

## How The Main Flows Fit Together

### Incident Flow

The incident flow is the best end-to-end orchestrated example in the repo:

```text
PagerDuty incident
  -> pagerduty-incident-analysis
  -> high-level Dynatrace sweep
  -> bounded child dynatrace-investigation work
  -> structured evidence packages
  -> final parent-page synthesis
```

Important rules:

- the parent incident workflow is the canonical writer for the parent Confluence page
- child Dynatrace investigations gather bounded evidence instead of narrating the whole incident
- `trial mode` is the default unless publishing is explicitly requested

### Incident Child Flow

The incident family uses an explicit parent-plus-child pattern:

```text
parent incident workflow
  -> high-level sweep
  -> small queue of bounded child tracks
  -> child evidence packages
  -> parent synthesis
  -> final incident page
```

Rules:

- the parent defines the exact question, time window, and scope for each child
- children stay within that scope and do not try to explain the whole incident
- children do not directly own the parent page
- the parent merges child findings into the final summary, root cause analysis, impact analysis, mitigations, and bottom-of-page evidence

Child investigations in this family should return:

- [templates/dynatrace-investigation-result.md](./templates/dynatrace-investigation-result.md)

That template is the bounded evidence contract for incident-style telemetry tracks.

### Service Analysis Flow

The service-analysis family now follows a cleaner pattern:

```text
service or component question
  -> service-analysis skill
  -> service-analysis-common
  -> branch-specific workflow or documentation-set workflow
  -> references/templates
  -> Confluence page, health summary, or small documentation set
```

Shared concerns such as environment defaults, evidence discipline, measurability caveats, and publishing standards live in the shared service-analysis layer instead of being copied into each skill.

### Service Analysis Child Flow

Service-analysis workflows are more selective about subagents, but when they split work they should also follow an explicit parent-plus-child pattern:

```text
parent service-analysis workflow
  -> optional bounded child tracks
  -> child evidence packages
  -> parent synthesis
  -> final page or health summary
```

Typical split patterns:

- code-path discovery vs telemetry-history analysis
- endpoint inventory vs Dynatrace entity mapping
- runtime structure vs interfaces and schemas
- telemetry topology vs operability guidance
- per-service health checks for a large owned-service set

Rules:

- the parent remains the canonical writer for the final artifact
- children return compact bounded evidence instead of ad hoc notes
- use subagents only when the tracks are independent and context reduction is actually worth it

Child investigations in this family should return:

- [templates/analysis-child-result.md](./templates/analysis-child-result.md)

That template is the default bounded evidence contract for non-incident analysis tracks.

### Jira Story Implementation Flow

```text
Jira URL or key
  -> full issue fields and every comment
  -> effective implementation contract
  -> repository and worktree preflight
  -> latest master
  -> Jira-keyed feature branch
  -> code and focused verification
  -> local implementation handoff
```

Rules:

- comment-sourced requirements remain visible in the effective contract
- an occupied feature checkout is preserved through a dedicated worktree, even when it is clean
- a worktree branch starts from freshly fetched `origin/master`
- commit, push, pull-request creation, and Jira mutation require an explicit follow-up request

### PR Review Flows

The review family now has three distinct flows because coaching, author-side triage, and reviewer-side review start from different objects and produce different outputs:

```text
GitHub username or author list
  -> pr-author-coaching
  -> sample recent authored PRs
  -> assess historical review and code patterns
  -> aggregate recurring strengths and issues
  -> produce coaching-ready guidance
```

```text
PR with human or AI comments
  -> babysit-pr
  -> triage current review state
  -> make safe revisions
  -> run narrow verification
  -> push branch updates
  -> reply to addressed review threads
```

```text
PR link or PR number
  -> review-pr
  -> read PR body
  -> read Jira story
  -> read epic when relevant
  -> read linked or discoverable Confluence design docs
  -> inspect current review state and diff
  -> produce findings in session
```

Rules:

- `pr-author-coaching` is author-history side and defaults to session-only coaching analysis.
- `babysit-pr` is author-side and defaults to active handling.
- `review-pr` is reviewer-side and defaults to a session-only review.
- `pr-author-coaching` may use bounded child sessions for larger samples, but the parent remains the canonical writer.
- `review-pr` should not comment on the PR or change code unless explicitly asked.
- `babysit-pr` may pull Jira, epic, or Confluence context when needed to answer a review comment correctly, but that is secondary to moving the PR forward.

## Repository Layout

- [codex/](./codex/)
  Thin Codex adapters. Each folder contains a `SKILL.md` and an `agents/openai.yaml` registration file.
- [claude/](./claude/)
  Thin Claude Code adapters. Each folder contains a `SKILL.md` that routes to the same shared workflows and support material.
- [workflows/](./workflows/)
  Shared operational procedures and branch playbooks.
- [tutorials/](./tutorials/)
  Guided first-run docs for common repo workflows.
- [references/](./references/)
  Stable routing rules, query patterns, interpretation guidance, and writing standards.
- [templates/](./templates/)
  Reusable artifact structures.
- [explanations/](./explanations/)
  Rationale, architecture, and decision-making guidance.
- [reviews/](./reviews/)
  Design and assessment artifacts that inform how the repo evolves.
- [scripts/](./scripts/)
  Setup and launcher utilities.

## Current Entry Skills

The shared workflow is canonical; the adapter columns show where each workflow is currently discoverable as a model-native skill.

| Capability | Codex adapter | Claude adapter | Primary outcome |
| --- | --- | --- | --- |
| PagerDuty incident analysis | [Codex](./codex/pagerduty-incident-analysis/) | [Claude](./claude/pagerduty-incident-analysis/) | Evidence-backed incident analysis and optional Confluence parent page |
| Dynatrace investigation | [Codex](./codex/dynatrace-investigation/) | [Claude](./claude/dynatrace-investigation/) | Rollout, incident, service-debugging, or exact-id telemetry investigation |
| PagerDuty assigned-service health | [Codex](./codex/pagerduty-assigned-service-health/) | [Claude](./claude/pagerduty-assigned-service-health/) | Health assessment for currently assigned services over an exact window |
| Incident follow-up planning | [Codex](./codex/incident-followup-planning/) | [Claude](./claude/incident-followup-planning/) | Validated Jira follow-up story set from incident evidence |
| Service system documentation | [Codex](./codex/service-system-documentation/) | [Claude](./claude/service-system-documentation/) | Diataxis-aligned overview, reference, and operability documentation |
| Service endpoint traffic analysis | [Codex](./codex/service-endpoint-traffic-analysis/) | [Claude](./claude/service-endpoint-traffic-analysis/) | Code-backed endpoint inventory mapped to production traffic |
| Service metric analysis | [Codex](./codex/service-metric-analysis/) | [Claude](./claude/service-metric-analysis/) | Emitted-metric inventory, telemetry trends, semantics, and caveats |
| Technical design documentation | [Codex](./codex/technical-design-documentation/) | [Claude](./claude/technical-design-documentation/) | Trial-mode design draft or explicitly published Confluence design |
| Implement Jira story | [Codex](./codex/implement-jira-story/) | [Claude](./claude/implement-jira-story/) | Local implementation and verification on a safe latest-master feature branch |
| APEX agent delegation | [Codex](./codex/apex-agent-delegation/) | [Claude](./claude/apex-agent-delegation/) | Bounded code-writing handoff that targets a draft PR |
| PR author coaching | [Codex](./codex/pr-author-coaching/) | [Claude](./claude/pr-author-coaching/) | Evidence-backed recurring strengths and issues across recent PRs |
| Babysit PR | [Codex](./codex/babysit-pr/) | [Claude](./claude/babysit-pr/) | Author-side review triage, safe revisions, verification, push, and replies |
| Review PR | [Codex](./codex/review-pr/) | [Claude](./claude/review-pr/) | Independent requirement-driven review, session-only by default |
| PR diagram | [Codex](./codex/pr-diagram/) | [Claude](./claude/pr-diagram/) | Concise PR body with one focused Mermaid diagram |
| Peer review | Not currently provided | [Claude](./claude/peer-review/) | Private FY26 performance-feedback draft with an evidence appendix |

## Current Workflows

### Shared Orchestrators And Common Layers

- [workflows/pagerduty-incident-analysis.md](./workflows/pagerduty-incident-analysis.md)
- [workflows/dynatrace-investigation.md](./workflows/dynatrace-investigation.md)
- [workflows/service-analysis-common.md](./workflows/service-analysis-common.md)
- [workflows/technical-design-documentation.md](./workflows/technical-design-documentation.md)
- [workflows/implement-jira-story.md](./workflows/implement-jira-story.md)
- [workflows/apex-agent-delegation.md](./workflows/apex-agent-delegation.md)
- [workflows/service-system-documentation.md](./workflows/service-system-documentation.md)
- [workflows/incident-followup-planning.md](./workflows/incident-followup-planning.md)
- [workflows/pr-author-coaching.md](./workflows/pr-author-coaching.md)
- [workflows/babysit-pr.md](./workflows/babysit-pr.md)
- [workflows/review-pr.md](./workflows/review-pr.md)
- [workflows/pr-diagram.md](./workflows/pr-diagram.md)
- [workflows/peer-review.md](./workflows/peer-review.md)

### Dynatrace Branch Workflows

- [workflows/dynatrace-rollout-check.md](./workflows/dynatrace-rollout-check.md)
- [workflows/dynatrace-incident-path-analysis.md](./workflows/dynatrace-incident-path-analysis.md)
- [workflows/dynatrace-service-debugging.md](./workflows/dynatrace-service-debugging.md)
- [workflows/dynatrace-guid-trace.md](./workflows/dynatrace-guid-trace.md)

### Service Analysis Branch Workflows

- [workflows/pagerduty-assigned-service-health.md](./workflows/pagerduty-assigned-service-health.md)
- [workflows/service-endpoint-traffic-analysis.md](./workflows/service-endpoint-traffic-analysis.md)
- [workflows/service-metric-analysis.md](./workflows/service-metric-analysis.md)

## Current Tutorials

- [tutorials/first-incident-investigation-trial-mode.md](./tutorials/first-incident-investigation-trial-mode.md)
  Guided first run for the PagerDuty incident workflow in `trial mode`.

## Current References

### Routing, Querying, And Interpretation

- [references/confluence-routing.md](./references/confluence-routing.md)
- [references/diataxis-writing-rules.md](./references/diataxis-writing-rules.md)
- [references/diataxis-review-checklist.md](./references/diataxis-review-checklist.md)
- [references/technical-design-writing-rules.md](./references/technical-design-writing-rules.md)
- [references/git-branch-naming.md](./references/git-branch-naming.md)
- [references/dynatrace-fast-path.md](./references/dynatrace-fast-path.md)
- [references/dynatrace-query-patterns.md](./references/dynatrace-query-patterns.md)
- [references/telemetry-measurability.md](./references/telemetry-measurability.md)
- [references/confluence-analysis-writing-standard.md](./references/confluence-analysis-writing-standard.md)
- [references/incident-analysis-surfaces.md](./references/incident-analysis-surfaces.md)
- [references/subagent-usage.md](./references/subagent-usage.md)

### Workflow-Specific Support

- [references/jira-incident-followup.md](./references/jira-incident-followup.md)
- [references/pr-coaching-rubric.md](./references/pr-coaching-rubric.md)
- [references/pr-review-context-gathering.md](./references/pr-review-context-gathering.md)
- [references/peer-review-rubric.md](./references/peer-review-rubric.md)
- [references/slack-setup.md](./references/slack-setup.md)
- [references/incident-investigation-lessons-2026-03-27.md](./references/incident-investigation-lessons-2026-03-27.md)

## Current Templates

- [templates/incident-analysis-page.md](./templates/incident-analysis-page.md)
  Parent incident document shape. This now converges on summary, root cause analysis, impact analysis, recommended solutions and mitigations, deployments or code references when relevant, and bottom-of-page evidence and queries.
- [templates/dynatrace-investigation-result.md](./templates/dynatrace-investigation-result.md)
  Child-result contract for incident-style Dynatrace investigations.
- [templates/analysis-child-result.md](./templates/analysis-child-result.md)
  Child-result contract for non-incident bounded analysis tracks such as service analysis and follow-up claim validation.
- [templates/service-overview-page.md](./templates/service-overview-page.md)
  Explanation-style overview template for what a service or system does, how it is shaped, and its main flows.
- [templates/service-reference-page.md](./templates/service-reference-page.md)
  Reference-style template for deployables, interfaces, schemas, runtime entities, and ownership anchors.
- [templates/service-operability-guide.md](./templates/service-operability-guide.md)
  How-to-style template for health checks, rollout validation, tracing, and common failure modes.
- [templates/tutorial-page.md](./templates/tutorial-page.md)
  Generic guided first-run template for teaching docs.
- [templates/how-to-page.md](./templates/how-to-page.md)
  Generic task-oriented template for procedural docs.
- [templates/reference-page.md](./templates/reference-page.md)
  Generic exact-lookup template for contracts, fields, and command docs.
- [templates/explanation-page.md](./templates/explanation-page.md)
  Generic rationale and tradeoff template for explanation-style architecture docs.
- [templates/technical-design-document.md](./templates/technical-design-document.md)
  Planned-change design-doc template covering goals, requirements, options, and transition plan.
- [templates/apex-agent-handoff.md](./templates/apex-agent-handoff.md)
  Launch-ready contract for bounded APEX code-writing delegation and required local follow-up.
- [templates/endpoint-traffic-analysis-page.md](./templates/endpoint-traffic-analysis-page.md)
  Final page shape for endpoint inventory and traffic analysis.
- [templates/service-metric-analysis-page.md](./templates/service-metric-analysis-page.md)
  Final page shape for service metric and telemetry analysis.
- [templates/incident-followup-story.md](./templates/incident-followup-story.md)
  Story-drafting template for incident follow-up work.
- [templates/peer-review-entry.md](./templates/peer-review-entry.md)
  Private evidence pack and two paste-ready FY26 performance-feedback entries.

## Current Explanations

- [explanations/repo-architecture.md](./explanations/repo-architecture.md)
  Why the repo is structured around shared workflows, references, templates, and thin skill adapters.
- [explanations/service-documentation-pattern.md](./explanations/service-documentation-pattern.md)
  Why service documentation should be produced as a Diataxis-aligned doc set, and how evidence, reference, and templates differ.
- [explanations/incident-analysis-family.md](./explanations/incident-analysis-family.md)
  How PagerDuty incident orchestration, bounded Dynatrace investigation, and follow-up planning fit together without duplicating ownership.
- [explanations/incident-analysis-pattern.md](./explanations/incident-analysis-pattern.md)
  Why the incident workflow defaults to `trial mode`, creates a parent surface early, uses bounded child tracks, and runs retrospective cleanup.
- [explanations/dynatrace-investigation-pattern.md](./explanations/dynatrace-investigation-pattern.md)
  Why the Dynatrace workflow uses a router, narrows entity scope early, prefers one branch, and keeps child investigations bounded.
- [explanations/dynatrace-evidence-interpretation.md](./explanations/dynatrace-evidence-interpretation.md)
  How to interpret ambiguous Dynatrace evidence such as low-load alerts, telemetry gaps, caller-vs-callee mismatch, and rollout correlation.

## Current Reviewed Artifacts

The `reviews/` tree preserves concrete outputs and design assessments without turning those one-off artifacts into reusable workflow instructions.

Design assessments:

- [reviews/design/dq-8931-extended-fraud-alert-handling-with-krm.md](./reviews/design/dq-8931-extended-fraud-alert-handling-with-krm.md)
- [reviews/design/payinz-lifecycle-automation.md](./reviews/design/payinz-lifecycle-automation.md)
- [reviews/design/skills-architecture-and-governance.md](./reviews/design/skills-architecture-and-governance.md)

Published service-documentation source sets:

- [reviews/service-docs/decisioning-services-compliance/](./reviews/service-docs/decisioning-services-compliance/)
  Compliance landing page, overview, reference, operability guide, AAN-generation deep dive, Mermaid sources, rendered snapshots, and Confluence publication manifest.
- [reviews/service-docs/provider-gateways-machine-learning-gateway/](./reviews/service-docs/provider-gateways-machine-learning-gateway/)
  Machine Learning Gateway overview, reference, operability guide, focused flow diagrams, and Confluence publication manifest.

Keep the Markdown and Mermaid source in git as the source of truth. Use each `confluence-manifest.yaml` for stable page ids and publication routing, and regenerate reader-facing diagram snapshots with [scripts/render-mermaid-diagrams](./scripts/render-mermaid-diagrams).

## Launcher And Maintenance Scripts

| Task | Codex | Claude Code |
| --- | --- | --- |
| Start a normal managed session | [scripts/codex-session](./scripts/codex-session) | [scripts/claude-session](./scripts/claude-session) |
| Start a preflighted incident session | [scripts/codex-incident-session](./scripts/codex-incident-session) | [scripts/claude-incident-session](./scripts/claude-incident-session) |
| Install local skill and shared-root links | [scripts/link-codex-skills](./scripts/link-codex-skills) | [scripts/link-claude-skills](./scripts/link-claude-skills) |
| Select the newest safe skill source and relink | [scripts/sync-codex-skills](./scripts/sync-codex-skills) | [scripts/sync-claude-skills](./scripts/sync-claude-skills) |
| Check for a CLI update | [scripts/update-codex](./scripts/update-codex) | [scripts/update-claude-code](./scripts/update-claude-code) |

Diagram snapshots are agent-independent and use [scripts/render-mermaid-diagrams](./scripts/render-mermaid-diagrams) with [mermaid.config.json](./mermaid.config.json).

## Using This Repo With Codex

The preferred setup is to symlink the repo-managed skills into your Codex home so new sessions automatically see repo updates.

Use:

```bash
./scripts/link-codex-skills
```

That script:

- links every folder under `codex/` into `${CODEX_HOME:-$HOME/.codex}/skills/`
- safely backs up conflicting copied folders with a timestamped suffix
- links the shared repo roots under `${CODEX_HOME:-$HOME/.codex}/` so relative references inside `SKILL.md` files keep working

The shared roots it links are:

- [workflows/](./workflows/)
- [references/](./references/)
- [templates/](./templates/)
- [reviews/](./reviews/)
- [scripts/](./scripts/)

Without those shared-root links, a skill folder may load its own `SKILL.md` but fail to resolve `../../workflows/...` or `../../templates/...`.

When you want a new session to automatically choose the newest available skill source, use:

```bash
./scripts/codex-session
```

That launcher:

- checks for a Codex CLI update at most once per day and installs it through OpenAI's official installer
- continues with the installed CLI if the update is temporarily unavailable
- refreshes the skill links before starting Codex
- prefers the current local checkout when it contains the newest edits or commits
- fast-forwards the local repo when the tracked upstream is newer and the pull is safe
- otherwise builds a temporary upstream snapshot and links from that without disturbing local work

To bypass the daily update cache when you know a new Codex release is available, use:

```bash
./scripts/update-codex --force
```

If you want to refresh links without starting Codex, use:

```bash
./scripts/sync-codex-skills
```

If you are intentionally iterating on local uncommitted workflow changes and want to force the current checkout as the source of truth, use:

```bash
./scripts/sync-codex-skills --link-only
```

## Using This Repo With Claude Code

Install Claude Code with Anthropic's recommended native installer:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Install the repo-managed skill links with:

```bash
./scripts/link-claude-skills
```

That script mirrors the Codex setup: it links every folder under `claude/` into `${CLAUDE_HOME:-$HOME/.claude}/skills/`, backs up conflicting folders, and links `workflows/`, `references/`, `templates/`, `reviews/`, and `scripts/` under the Claude home so shared relative references resolve.

For a managed session, use:

```bash
./scripts/claude-session
```

The `claude-session` and `claude-incident-session` launchers:

- promote the native binary at `${CLAUDE_CODE_BIN_DIR:-$HOME/.local/bin}/claude`
- check for an update at most once per day with `claude update`
- attempt the official native installer when that binary is not installed yet
- keep an already installed native CLI usable when a temporary update fails
- select the newest safe local or tracked-upstream skill source and refresh links
- expose this repo to the launched Claude Code session with `--add-dir`

To bypass the daily update cache, use:

```bash
./scripts/update-claude-code --force
```

If you want to refresh Claude skill links without starting Claude Code, use:

```bash
./scripts/sync-claude-skills
```

When intentionally testing uncommitted workflow changes, force the current checkout as the link source with:

```bash
./scripts/sync-claude-skills --link-only
```

## Incident Session Launchers

Both agent adapters provide a dedicated incident launcher:

- `codex-incident-session trial`
- `codex-incident-session publish`
- `claude-incident-session trial`
- `claude-incident-session publish`

The scripts live at:

- [scripts/codex-incident-session](./scripts/codex-incident-session)
- [scripts/claude-incident-session](./scripts/claude-incident-session)

Both launchers:

- update the native CLI and refresh the corresponding skill links
- start the selected agent in the target workspace with this repo available
- preflight harmless PagerDuty and Dynatrace reads before a real incident id is supplied
- keep preflight read-only in both modes; `trial` skips routine Atlassian access while `publish` warms one harmless Atlassian read
- stop after preflight and wait for the incident id, reducing approval interruptions during the investigation

By default, each launcher uses the current directory as the target workspace. To use another workspace, pass it as the second argument:

- `codex-incident-session trial /path/to/services`
- `claude-incident-session trial /path/to/services`

You can also set `CODEX_INCIDENT_WORKDIR` or `CLAUDE_INCIDENT_WORKDIR` for the corresponding launcher.

## Example Prompts

- `Use $pagerduty-incident-analysis to investigate PagerDuty incident Q2YLVYYF9DVCJK in trial mode.`
- `Use $pagerduty-incident-analysis to analyze PagerDuty incident Q2YLVYYF9DVCJK and publish the parent Confluence write-up.`
- `Use $dynatrace-investigation to determine whether a deployment rollout degraded decision-engine in production.`
- `Use $dynatrace-investigation to trace this GUID through logs, spans, and events and tell me where the trail stops.`
- `Use $pagerduty-assigned-service-health to assess the health of my currently assigned PagerDuty services for last weekend.`
- `Use $service-endpoint-traffic-analysis to analyze risk-manager endpoint traffic and create or update the Confluence page.`
- `Use $service-metric-analysis to inspect a repo service, analyze its emitted metrics in Dynatrace, and publish the findings to Confluence.`
- `Use $incident-followup-planning to validate the incident page and create follow-up Jira stories under the incident epic.`
- `Use $technical-design-documentation to draft a trial-mode design for DQ-1234 from the Jira requirements and current code.`
- `Use $implement-jira-story to implement https://quadpay.atlassian.net/browse/DQ-9407 in quadpay-services, including all acceptance criteria and comments.`
- `Use $apex-agent-delegation to prepare and launch a bounded code-writing handoff for DQ-1234 that opens a draft PR.`
- `Use $pr-author-coaching to analyze davidsgbang's last 5 PRs in quadpay/quadpay-services and tell me the recurring issues and strengths to coach on.`
- `Use $babysit-pr to handle the review comments on my PR, make any needed code changes, push updates, and reply to the threads.`
- `Use $review-pr to review this PR against the linked Jira story, epic if needed, Confluence design docs, and the current code, then discuss the findings in session.`
- `Use $pr-diagram to rewrite this PR description around the one changed interaction reviewers need to understand.`
- `Use $peer-review to draft my FY26 feedback for this Zipster from Jira, GitHub, and the observations I provide.` (Claude Code only)

## Adding New Capabilities

Use these rules before adding more repo surface area.

### Create a New Skill When

- the starting object is materially different
- the output contract is materially different
- the mode of operation is materially different
- folding the trigger into an existing skill would make the entrypoint confusing

### Create a New Workflow When

- the reusable procedure is different enough that one shared workflow becomes unclear

### Create a Branch Workflow When

- the top-level flow is the same
- one phase changes significantly by scenario

This is the model used by `dynatrace-investigation` and now by the service-analysis family.

### Create a New Reference When

- the information is stable
- multiple workflows need it
- it is fact-heavy or pattern-heavy rather than procedural

### Create a New Template When

- multiple workflows emit a similar artifact shape
- the output structure is worth standardizing independently of the procedure

### Create a Script When

- the step is brittle, deterministic, or repeatedly reimplemented
- a script materially improves reliability rather than just saving tokens

## Portability And Maintenance Rules

- Keep reusable operating logic in `workflows/`, `references/`, and `templates/`.
- Keep skill wrappers thin.
- Do not store secrets or credentials in the repo.
- Default to exact timestamps and explicit ids.
- Prefer narrow, defensible scopes over broad scans.
- State caveats in the artifact itself, not only in chat.
- Distinguish direct evidence from interpretation.
- If telemetry cannot support a question, say so explicitly instead of inferring beyond the signal.

## Additional Adapter Targets

Codex and Claude Code are current adapters. The shared layout can support additional thin adapters later, for example:

- `chatgpt/`
- `cursor/`
