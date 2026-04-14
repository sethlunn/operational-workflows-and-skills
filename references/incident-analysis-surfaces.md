# Reference

- Subject: Incident analysis workflow family
- Scope: PagerDuty incident orchestration, Dynatrace child investigations, parent and child output contracts, and post-incident follow-up planning
- Canonical owner: `workflows/pagerduty-incident-analysis.md`
- Last reviewed: `2026-04-14`

# Definitions

- Parent incident workflow:
  - `workflows/pagerduty-incident-analysis.md`
  - The canonical orchestrator for a PagerDuty-backed incident investigation.
- Parent investigation surface:
  - The canonical incident artifact owned by the parent workflow.
  - In `publish mode`, this is the Confluence incident page.
  - In `trial mode`, this is an in-memory parent-page-ready result.
- Child investigation:
  - One bounded Dynatrace track that answers one exact question in one time window for one scoped path, dependency, or identifier chain.
- Trial mode:
  - Full incident investigation without creating or updating the Confluence page.
- Publish mode:
  - Full incident investigation with early parent-page creation or update in Confluence.
- Draft mode:
  - Follow-up-planning mode that validates conclusions and drafts Jira work without writing Jira issues.
- Write mode:
  - Follow-up-planning mode that validates conclusions and then creates or updates Jira issues when explicitly requested.
- Retrospective cleanup:
  - The final normalization pass that refreshes status, converts timestamps to exact absolute times, removes stale live-incident wording, and rewrites the top summary in final tense.

# Contracts Or Structure

## Entry Surfaces

- `codex/pagerduty-incident-analysis/SKILL.md`
  - Entry skill for end-to-end incident analysis from PagerDuty through Dynatrace to a Confluence-ready result.
- `codex/dynatrace-investigation/SKILL.md`
  - Entry skill for reusable Dynatrace investigations, including bounded child tracks under the parent incident workflow.
- `codex/incident-followup-planning/SKILL.md`
  - Entry skill for validating an incident page and turning the validated conclusions into Jira follow-up work.

## Shared Workflows

- `workflows/pagerduty-incident-analysis.md`
  - Parent orchestrator.
  - Resolves PagerDuty metadata, ownership, time window, high-level Dynatrace sweep, child-track queue, synthesis, and cleanup.
- `workflows/dynatrace-investigation.md`
  - Dynatrace router.
  - Chooses exactly one branch playbook and enforces narrow scoping.
- `workflows/dynatrace-incident-path-analysis.md`
  - Default Dynatrace branch for high-level incident sweeps and for narrow service-path or dependency-path child tracks.
- `workflows/incident-followup-planning.md`
  - Post-incident validation and Jira follow-up workflow.

## Output Contracts

- `templates/incident-analysis-page.md`
  - Parent incident artifact contract.
  - Defines the expected sections for summary, timeline, mapping, root cause, impact, mitigations, investigation queue, deployments or code references, and exact evidence.
- `templates/dynatrace-investigation-result.md`
  - Child investigation contract.
  - Requires exact question, exact scope, direct evidence, interpretation, confidence, unresolved gap, and next narrow query.
- `templates/incident-followup-story.md`
  - Follow-up story draft contract for Jira work under the incident epic.

## Default Modes And Ownership Rules

- `pagerduty-incident-analysis` defaults to `trial mode`.
- `incident-followup-planning` defaults to `draft mode`.
- The parent incident workflow is the canonical writer for the parent incident page or parent-page-ready result.
- Child Dynatrace investigations return bounded evidence packages and do not own the main incident narrative.
- External writes require explicit user intent:
  - Confluence writes for the incident page
  - Jira writes for follow-up planning

## Investigation Window Defaults

- Default start:
  - `PagerDuty incident created time - 30 minutes`
- Default end:
  - `PagerDuty incident resolved time + 15 minutes`
- If the incident is still open:
  - use the current time as the end
- Expand the window only when alerts, notes, or change events justify it.
- Normalize the final window into exact absolute timestamps in the user's timezone.

## Dynatrace Branch Mapping

- `workflows/dynatrace-incident-path-analysis.md`
  - Use for high-level incident sweeps and narrow service-path or dependency-path tracks.
- `workflows/dynatrace-service-debugging.md`
  - Use for a specific failing endpoint, worker path, exception family, or dependency failure.
- `workflows/dynatrace-guid-trace.md`
  - Use for exact identifier tracing or missing-event questions.
- `workflows/dynatrace-rollout-check.md`
  - Use for deployment-correlation questions.

## Required Child-Investigation Fields

- Exact question answered
- Exact time window searched
- Exact scoped entities, dependencies, objects, or fields
- Exact queries that materially support the result
- Direct evidence
- Interpretation
- Confidence
- Strongest unresolved gap
- Next best narrow query if unresolved

# Examples

- Minimal incident investigation path:
  - `codex/pagerduty-incident-analysis/SKILL.md`
  - `workflows/pagerduty-incident-analysis.md`
  - high-level Dynatrace sweep through `workflows/dynatrace-investigation.md`
  - bounded child evidence packages using `templates/dynatrace-investigation-result.md`
  - parent-page-ready result or published Confluence page using `templates/incident-analysis-page.md`

- Post-incident follow-up path:
  - existing incident page and RAC or summary
  - `codex/incident-followup-planning/SKILL.md`
  - `workflows/incident-followup-planning.md`
  - story drafts or Jira writes using `templates/incident-followup-story.md`

# Constraints And Caveats

- The incident family assumes PagerDuty is the starting ownership and timing surface unless evidence clearly proves otherwise.
- Duplicate PagerDuty incidents, composite environment alerts, and routing noise are normal cases handled by the parent workflow.
- The high-level Dynatrace sweep is for queue shaping, not for proving every root-cause claim on its own.
- `trial mode` is not a shallow mode. It performs the full investigation without external writes.
- Follow-up planning validates an existing incident artifact and a second analysis. It is not the preferred starting point for raw incident triage.

# Related Docs

- How-to:
  - `workflows/pagerduty-incident-analysis.md`
  - `workflows/incident-followup-planning.md`
- Explanation:
  - `explanations/incident-analysis-pattern.md`
  - `explanations/incident-analysis-family.md`
- Tutorial:
  - `tutorials/first-incident-investigation-trial-mode.md`
