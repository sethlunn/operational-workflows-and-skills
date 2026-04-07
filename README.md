# Operational Workflows and Skills

Portable, company-specific operational workflows and reusable skill adapters for LLM agents.

This repo keeps the reusable operating model in shared Markdown and keeps model-specific skill wrappers thin. The goal is to make one investigation or analysis pattern reusable across Codex today and other agent environments later.

Start with [START-HERE.md](./START-HERE.md).

## Use This Repo By Need

Diataxis is the simplest useful lens for this repo: choose documentation based on what the reader needs right now.

### Tutorials

Use tutorials when the goal is to learn safely through a guided first run.

Current tutorial:

- `tutorials/first-incident-investigation-trial-mode.md`

### How-to Guides

Use how-to guides when the goal is to complete an operational task.

The main how-to surface is `workflows/`.

Good starting points:

- `workflows/pagerduty-incident-analysis.md`
- `workflows/dynatrace-investigation.md`
- `workflows/pagerduty-assigned-service-health.md`
- `workflows/service-endpoint-traffic-analysis.md`
- `workflows/service-metric-analysis.md`
- `workflows/incident-followup-planning.md`
- `workflows/babysit-pr.md`
- `workflows/review-pr.md`

### Reference

Use reference when you need exact facts, patterns, or output contracts.

The main reference surface is:

- `references/`
- `templates/`
- `scripts/`

High-value reference files:

- `references/diataxis-writing-rules.md`
- `references/diataxis-review-checklist.md`
- `references/confluence-routing.md`
- `references/dynatrace-query-patterns.md`
- `references/dynatrace-fast-path.md`
- `references/subagent-usage.md`
- `templates/tutorial-page.md`
- `templates/how-to-page.md`
- `templates/reference-page.md`
- `templates/explanation-page.md`
- `templates/incident-analysis-page.md`
- `templates/dynatrace-investigation-result.md`
- `templates/analysis-child-result.md`

### Explanation

Use explanation when you need rationale, tradeoffs, or the mental model behind the workflows.

Start with:

- `explanations/repo-architecture.md`
- `explanations/incident-analysis-pattern.md`
- `explanations/dynatrace-investigation-pattern.md`
- `reviews/design/skills-architecture-and-governance.md`

Some explanation-oriented material still lives under `references/` and should move over time:

- `references/dynatrace-evidence-interpretation.md`
- `references/incident-investigation-lessons-2026-03-27.md`

## Core Model

This repo is organized around a small set of layers:

1. `skills`
   Thin entrypoints that tell an agent when to trigger and which shared workflow to read.
2. `workflows`
   The real reusable procedure.
3. `references`
   Stable facts, query patterns, routing rules, and caveat handling.
4. `templates`
   Output structure for Confluence pages and other reusable artifacts.
5. `reviews`
   Design, code, and planning assessments.
6. `scripts`
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

- Keep `SKILL.md` files small and routing-oriented.
- Put reusable business logic in `workflows/`, not in skill wrappers.
- Put stable supporting knowledge in `references/`, not in workflows unless it is procedural.
- Put output shape in `templates/`, not process.
- Add scripts only when a step is brittle or benefits from deterministic execution.
- Prefer exact dates, exact ids, and exact query shapes over vague summaries.
- Separate direct evidence from interpretation in every analysis workflow.
- Use one canonical writer for any published parent artifact.

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

- `service-endpoint-traffic-analysis`
  Inspect code, inventory endpoints, map them to Dynatrace traffic, and publish a cleanup-oriented Confluence page.
- `service-metric-analysis`
  Inspect code, identify emitted metrics, analyze telemetry history and breakdowns, and publish a Confluence page.
- `pagerduty-assigned-service-health`
  Discover the current user's PagerDuty-owned services, map them to Dynatrace production entities, and summarize health over an exact time window.

These now share a common service-analysis layer:

- `workflows/service-analysis-common.md`
- `references/telemetry-measurability.md`
- `references/confluence-analysis-writing-standard.md`

### Review And Triage

- `babysit-pr`
  Active code-owner workflow for triaging review comments, making safe revisions, pushing updates, and replying to review threads.
- `review-pr`
  Requirement-driven PR review workflow that reads Jira, epic, and Confluence context before reviewing the code and discussing findings in session.

These are intentionally separate:

- `babysit-pr`
  Use when you own the PR and want the agent to move it forward by handling review feedback.
- `review-pr`
  Use when you want an independent review of the PR against the story, design context, and current code.

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

- `templates/dynatrace-investigation-result.md`

That template is the bounded evidence contract for incident-style telemetry tracks.

### Service Analysis Flow

The service-analysis family now follows a cleaner pattern:

```text
service or component question
  -> service-analysis skill
  -> service-analysis-common
  -> branch-specific workflow
  -> references/templates
  -> Confluence page or health summary
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
- per-service health checks for a large owned-service set

Rules:

- the parent remains the canonical writer for the final artifact
- children return compact bounded evidence instead of ad hoc notes
- use subagents only when the tracks are independent and context reduction is actually worth it

Child investigations in this family should return:

- `templates/analysis-child-result.md`

That template is the default bounded evidence contract for non-incident analysis tracks.

### PR Review Flows

The review family now has two distinct flows because the author-side and reviewer-side jobs are materially different:

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

- `babysit-pr` is author-side and defaults to active handling.
- `review-pr` is reviewer-side and defaults to a session-only review.
- `review-pr` should not comment on the PR or change code unless explicitly asked.
- `babysit-pr` may pull Jira, epic, or Confluence context when needed to answer a review comment correctly, but that is secondary to moving the PR forward.

## Repository Layout

- `codex/`
  Thin Codex adapters. Each folder contains a `SKILL.md` and usually `agents/openai.yaml`.
- `workflows/`
  Shared operational procedures and branch playbooks.
- `tutorials/`
  Guided first-run docs for common repo workflows.
- `references/`
  Stable routing rules, query patterns, interpretation guidance, and writing standards.
- `templates/`
  Reusable artifact structures.
- `explanations/`
  Rationale, architecture, and decision-making guidance.
- `reviews/`
  Design and assessment artifacts that inform how the repo evolves.
- `scripts/`
  Setup and launcher utilities.

## Current Entry Skills

- `codex/pagerduty-incident-analysis`
- `codex/dynatrace-investigation`
- `codex/pagerduty-assigned-service-health`
- `codex/service-endpoint-traffic-analysis`
- `codex/service-metric-analysis`
- `codex/incident-followup-planning`
- `codex/babysit-pr`
- `codex/review-pr`

## Current Workflows

### Shared Orchestrators And Common Layers

- `workflows/pagerduty-incident-analysis.md`
- `workflows/dynatrace-investigation.md`
- `workflows/service-analysis-common.md`
- `workflows/incident-followup-planning.md`
- `workflows/babysit-pr.md`
- `workflows/review-pr.md`

### Dynatrace Branch Workflows

- `workflows/dynatrace-rollout-check.md`
- `workflows/dynatrace-incident-path-analysis.md`
- `workflows/dynatrace-service-debugging.md`
- `workflows/dynatrace-guid-trace.md`

### Service Analysis Branch Workflows

- `workflows/pagerduty-assigned-service-health.md`
- `workflows/service-endpoint-traffic-analysis.md`
- `workflows/service-metric-analysis.md`

## Current Tutorials

- `tutorials/first-incident-investigation-trial-mode.md`
  Guided first run for the PagerDuty incident workflow in `trial mode`.

## Current References

### Routing, Querying, And Interpretation

- `references/confluence-routing.md`
- `references/diataxis-writing-rules.md`
- `references/diataxis-review-checklist.md`
- `references/dynatrace-fast-path.md`
- `references/dynatrace-query-patterns.md`
- `references/dynatrace-evidence-interpretation.md`
- `references/telemetry-measurability.md`
- `references/confluence-analysis-writing-standard.md`
- `references/subagent-usage.md`

### Workflow-Specific Support

- `references/jira-incident-followup.md`
- `references/pr-review-context-gathering.md`
- `references/slack-setup.md`
- `references/incident-investigation-lessons-2026-03-27.md`

## Current Templates

- `templates/incident-analysis-page.md`
  Parent incident document shape. This now converges on summary, root cause analysis, impact analysis, recommended solutions and mitigations, deployments or code references when relevant, and bottom-of-page evidence and queries.
- `templates/dynatrace-investigation-result.md`
  Child-result contract for incident-style Dynatrace investigations.
- `templates/analysis-child-result.md`
  Child-result contract for non-incident bounded analysis tracks such as service analysis and follow-up claim validation.
- `templates/tutorial-page.md`
  Generic guided first-run template for teaching docs.
- `templates/how-to-page.md`
  Generic task-oriented template for procedural docs.
- `templates/reference-page.md`
  Generic exact-lookup template for contracts, fields, and command docs.
- `templates/explanation-page.md`
  Generic rationale and tradeoff template for architecture and design docs.
- `templates/endpoint-traffic-analysis-page.md`
  Final page shape for endpoint inventory and traffic analysis.
- `templates/service-metric-analysis-page.md`
  Final page shape for service metric and telemetry analysis.
- `templates/incident-followup-story.md`
  Story-drafting template for incident follow-up work.

## Current Explanations

- `explanations/repo-architecture.md`
  Why the repo is structured around shared workflows, references, templates, and thin skill adapters.
- `explanations/incident-analysis-pattern.md`
  Why the incident workflow defaults to `trial mode`, creates a parent surface early, uses bounded child tracks, and runs retrospective cleanup.
- `explanations/dynatrace-investigation-pattern.md`
  Why the Dynatrace workflow uses a router, narrows entity scope early, prefers one branch, and keeps child investigations bounded.

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

- `workflows/`
- `references/`
- `templates/`
- `reviews/`
- `scripts/`

Without those shared-root links, a skill folder may load its own `SKILL.md` but fail to resolve `../../workflows/...` or `../../templates/...`.

## Incident Session Launcher

There is a dedicated incident launcher:

- `codex-incident-session trial`
- `codex-incident-session publish`

The script lives at:

- `scripts/codex-incident-session`

What it does:

- starts Codex in the target workspace
- adds this repo as an extra writable directory
- warms likely PagerDuty and Dynatrace reads early
- reduces approval friction during live incident work

By default, the launcher uses your current directory as the target workspace. If you want a different workspace, pass it explicitly:

- `codex-incident-session trial /path/to/services`

You can also set `CODEX_INCIDENT_WORKDIR`.

## Example Prompts

- `Use $pagerduty-incident-analysis to investigate PagerDuty incident Q2YLVYYF9DVCJK in trial mode.`
- `Use $pagerduty-incident-analysis to analyze PagerDuty incident Q2YLVYYF9DVCJK and publish the parent Confluence write-up.`
- `Use $dynatrace-investigation to determine whether a deployment rollout degraded decision-engine in production.`
- `Use $dynatrace-investigation to trace this GUID through logs, spans, and events and tell me where the trail stops.`
- `Use $pagerduty-assigned-service-health to assess the health of my currently assigned PagerDuty services for last weekend.`
- `Use $service-endpoint-traffic-analysis to analyze risk-manager endpoint traffic and create or update the Confluence page.`
- `Use $service-metric-analysis to inspect a repo service, analyze its emitted metrics in Dynatrace, and publish the findings to Confluence.`
- `Use $incident-followup-planning to validate the incident page and create follow-up Jira stories under the incident epic.`
- `Use $babysit-pr to handle the review comments on my PR, make any needed code changes, push updates, and reply to the threads.`
- `Use $review-pr to review this PR against the linked Jira story, epic if needed, Confluence design docs, and the current code, then discuss the findings in session.`

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

## Future Adapters

The shared layout is intended to support additional adapters later, for example:

- `claude/`
- `chatgpt/`
- `cursor/`
