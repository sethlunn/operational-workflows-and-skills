# Operational Workflows and Skills

Portable, company-specific operational workflows and reusable skill adapters for LLM agents.

This repo keeps the business logic in shared Markdown so the same operating model can be reused across agent systems, with thin model-specific wrappers on top.

## What This Repo Is

This repo is a small operational playbook system for AI agents.

It is designed around four ideas:

- shared workflows hold the real procedure
- shared references keep routing and environment details stable
- shared reviews capture design, code, and plan assessments
- shared templates keep output consistent
- model-specific adapters stay thin

That split lets one investigation pattern be reused across Codex today and other agent environments later.

## Demo Story

The best demo flow in the repo today is the PagerDuty incident analysis workflow.

That workflow now acts as an orchestrator:

1. Resolve the PagerDuty incident, owning service, and exact incident window.
2. Run a high-level Dynatrace sweep over the incident alert surface.
3. Create the parent investigation surface early:
   - parent Confluence page in `publish mode`
   - parent-page-ready outline in `trial mode`
4. Turn the high-level sweep into a small queue of narrow child investigations.
5. Run deeper Dynatrace investigations on specific scopes such as:
   - one service path
   - one downstream database or storage path
   - one third-party or internal dependency path
   - one identifier or propagation path
6. Require each child investigation to return a structured evidence package.
7. Synthesize those child results into a final cross-investigation assessment.
8. Update the parent investigation surface with the stitched result.

The important operating rule is:

- the parent incident workflow is the canonical writer for the parent Confluence page

Child investigations gather bounded evidence. They do not each try to narrate the whole incident.

For live demos and trial runs, the workflow should also feel low-friction:

- default to `trial mode` unless publishing is explicitly requested
- execute routine read-only local inspection directly
- execute PagerDuty, Dynatrace, and Atlassian read operations directly
- only interrupt when an external write, destructive action, or sandbox escalation is actually required

## Current Skill Suite

### `pagerduty-incident-analysis`

The end-to-end incident orchestrator.

Use it when the starting point is a PagerDuty incident id or incident URL and the goal is to produce a solid investigation write-up in Confluence.

It combines:

- PagerDuty incident resolution
- service ownership and routing
- high-level Dynatrace triage
- bounded child investigations
- synthesis into a parent Confluence document

### `dynatrace-investigation`

The core Dynatrace router and evidence-gathering playbook.

Use it for:

- rollout checks
- incident investigations
- direct debugging
- GUID and data-validation tracing

Internally, it now routes into smaller branch docs instead of one large workflow.

This workflow can run either as:

- a standalone investigation skill
- a bounded child investigation inside the PagerDuty incident workflow

### `pagerduty-assigned-service-health`

The on-call health rollup skill.

Use it to discover the current user's PagerDuty-owned services, map them to Dynatrace production entities, and summarize service health over an exact time window.

### `service-endpoint-traffic-analysis`

The service inventory and traffic-mapping skill.

Use it to inspect a repo service, inventory its HTTP endpoints from code, map those endpoints to Dynatrace traffic, and publish a Confluence page with usage tiers and cleanup candidates.

### `incident-followup-planning`

The post-incident validation and Jira follow-up skill.

Use it when an incident already has a Confluence page, RAC, or another agent's assessment and the goal is to validate the claims, tighten the final assessment, and create actionable follow-up stories under an incident epic.

It combines:

- Confluence and RAC validation
- PagerDuty, Dynatrace, and code-backed claim checks
- actionable story breakdown
- Jira field-aware story creation
- optional epic cleanup and incident-channel summary

### `babysit-pr`

The PR review-triage skill.

It is not part of the Dynatrace/PagerDuty investigation flow, but it lives in the same repo because it follows the same pattern of reusable operational workflow plus thin adapter.

## How The Pieces Fit Together

For incident work, the relationships are:

- `pagerduty-incident-analysis` is the orchestrator
- `dynatrace-investigation` is the worker playbook
- `incident-analysis-page.md` is the parent page structure
- `dynatrace-investigation-result.md` is the child investigation contract
- `confluence-routing.md` controls where the page should live
- `dynatrace-query-patterns.md` provides starter query shapes for targeted evidence gathering

In practice, the flow looks like this:

```text
PagerDuty incident
  -> parent incident workflow
  -> high-level Dynatrace sweep
  -> parent investigation surface
  -> bounded child Dynatrace investigations
  -> structured child evidence packages
  -> cross-investigation synthesis
  -> finalized parent result
```

The operating modes are:

- `trial mode`
  Investigate fully, return a parent-page-ready result, and do not publish to Confluence.
- `publish mode`
  Run the same investigation and create or update the Confluence page as part of the workflow.

## Incident Session Launcher

There is a dedicated launcher for incident investigations:

- `codex-incident-session trial`
- `codex-incident-session publish`

The launcher script lives at:

- `scripts/codex-incident-session`

The PATH command is installed as:

- `~/.local/bin/codex-incident-session`

What it does:

- starts Codex in the trusted `quadpay-services/services` workspace
- adds this workflows repo as an extra writable directory
- starts in interactive mode with the right sandbox posture
- runs a short preflight prompt before any real incident work
- intentionally warms the likely PagerDuty and Dynatrace reads, including Dynatrace `execute_dql`

Why this helps:

- the first MCP approval prompts happen at session start instead of in the middle of an incident run
- you can choose `Allow for this session` once during preflight
- after preflight completes, the session waits for the incident id

## Repository Layout

- `workflows/`
  Shared operational procedures written to be readable by any model.
- `references/`
  Stable routing, query-pattern, naming, and environment-specific reference material.
- `reviews/`
  Opinionated review artifacts such as design reviews, code review notes, and planning assessments.
- `templates/`
  Reusable output templates for parent pages, child investigation results, and endpoint analysis pages.
- `codex/`
  Thin Codex adapters that wrap the shared workflow docs as `SKILL.md` skills.
- `scripts/`
  Helper launchers for repeatable investigation sessions.

## Current Workflows

- `workflows/pagerduty-incident-analysis.md`
  Orchestrated PagerDuty-to-Dynatrace-to-Confluence incident workflow.
- `workflows/dynatrace-investigation.md`
  Shared Dynatrace router with preflight, child-investigation contract, and output rules.
- `workflows/dynatrace-rollout-check.md`
  Deployment and rollout regression playbook.
- `workflows/dynatrace-incident-path-analysis.md`
  Incident-path and dependency-path investigation playbook.
- `workflows/dynatrace-service-debugging.md`
  Direct service and failure-path debugging playbook.
- `workflows/dynatrace-guid-trace.md`
  Identifier tracing and missing-event validation playbook.
- `workflows/pagerduty-assigned-service-health.md`
  PagerDuty ownership discovery plus Dynatrace health assessment.
- `workflows/service-endpoint-traffic-analysis.md`
  Code-derived endpoint inventory plus Dynatrace traffic analysis.
- `workflows/incident-followup-planning.md`
  Validate incident documents and external analysis, then create follow-up Jira stories under an epic.
- `workflows/babysit-pr.md`
  PR review triage and signed reply workflow.

## Templates That Matter For The Demo

- `templates/incident-analysis-page.md`
  Parent Confluence page for incident work, including initial triage, investigation queue, parallel investigations, and cross-investigation assessment.
- `templates/dynatrace-investigation-result.md`
  Structured return contract for child Dynatrace investigations.
- `templates/endpoint-traffic-analysis-page.md`
  Confluence page structure for endpoint traffic analysis.

## Dynatrace Breakdown

The Dynatrace skill now uses a thin shared router plus branch-specific docs.

- use `workflows/dynatrace-investigation.md` for shared preflight, scope discipline, child-investigation rules, and output rules
- then read only the branch you need
- use `references/dynatrace-evidence-interpretation.md` when the evidence is ambiguous, especially for:
  - caller-vs-callee mismatch
  - telemetry gaps
  - rollout correlation vs code causation
  - late secondary events vs onset

## Why The Incident Workflow Changed

The incident workflow used to be mostly linear.

It now supports a more realistic operating model:

- broad incident triage first
- deeper investigations second
- one canonical parent document
- explicit separation between direct evidence and interpretation
- synthesis across multiple narrow investigations instead of one sprawling scan

That makes the workflow better for real incidents and better for demos, because it shows:

- orchestration
- agent specialization
- controlled parallelism
- auditable evidence gathering
- structured final synthesis

## Codex Usage

To use a skill with Codex, copy or symlink one of the folders under `codex/` into `~/.codex/skills/`.

For this repo, the preferred setup is to symlink the repo-managed skill folders so new Codex sessions automatically see repo updates. Use:

```bash
/Users/sethlunn/RiderProjects/operational-workflows-and-skills/scripts/link-codex-skills
```

That script links every folder under `codex/` into `~/.codex/skills/` and safely backs up any existing copied folders with a timestamped `.bak.<timestamp>` suffix. Once that is done, updating this repo updates the skills for future Codex sessions without another sync step. If you add a brand-new skill folder later, rerun the script once so the new folder gets linked too.

It also links the shared repo roots into `~/.codex/`:

- `workflows/`
- `references/`
- `templates/`
- `reviews/`
- `scripts/`

That matters because the `SKILL.md` files intentionally reference shared docs like `../../workflows/...` and `../../templates/...`. Without those shared-root links, copied or symlinked skill folders can load `SKILL.md` itself but fail to resolve the supporting materials.

Good demo prompts:

- `codex-incident-session trial`
- `codex-incident-session publish`
- `Use $pagerduty-incident-analysis to investigate PagerDuty incident Q2YLVYYF9DVCJK in trial mode. Do not publish anything unless I ask.`
- `Use $pagerduty-incident-analysis to analyze PagerDuty incident Q2YLVYYF9DVCJK, run deeper Dynatrace investigations as needed, and publish the parent Confluence write-up.`
- `Use $dynatrace-investigation to determine whether a deployment rollout degraded decision-engine in production.`
- `Use $dynatrace-investigation to trace this GUID through logs, spans, and events and tell me where the trail stops.`
- `Use $pagerduty-assigned-service-health to assess the health of my currently assigned PagerDuty services for last weekend.`
- `Use $service-endpoint-traffic-analysis to analyze risk-manager endpoint traffic and create or update the Confluence page.`

## Portability Rules

- Keep business logic in `workflows/`, `references/`, and `templates/`.
- Keep review artifacts in `reviews/`, organized by review type such as `design/`, `code/`, or `plans/`.
- Keep model-specific files as thin wrappers only.
- Avoid embedding secrets, tokens, or environment credentials in the repo.
- Prefer exact dates, entity ids, and query snippets over vague summaries.
- Prefer narrow, defensible scopes over broad scans when investigating operational issues.

## Next Adapters

This layout is meant to support additional adapters later, for example:

- `claude/`
- `chatgpt/`
- `cursor/`
