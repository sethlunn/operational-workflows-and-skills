---
name: pagerduty-incident-analysis
description: "Analyze a PagerDuty incident from an incident id or PagerDuty incident URL, inspect the incident and impacted service through PagerDuty MCP, investigate the service and incident window in Dynatrace, and create or update a Confluence incident analysis page. Use when the user asks to investigate a PagerDuty incident such as Q2YLVYYF9DVCJK or a https://zipcous.pagerduty.com/incidents/... link and wants a write-up in Confluence."
---

# PagerDuty Incident Analysis

Read [../../workflows/pagerduty-incident-analysis.md](../../workflows/pagerduty-incident-analysis.md) before starting.
Read [../../references/confluence-routing.md](../../references/confluence-routing.md) before choosing a Confluence destination.
Read [../../templates/incident-analysis-page.md](../../templates/incident-analysis-page.md) when creating or updating a Confluence page.
Read [../../templates/dynatrace-investigation-result.md](../../templates/dynatrace-investigation-result.md) when coordinating child Dynatrace investigations.

Follow the shared workflow, default to `trial mode` unless the user explicitly asks to publish, preserve the timestamp-prefixed page-title rule in `publish mode`, and keep the parent incident workflow as the canonical writer for the parent page.

Do not interrupt for routine read-only local inspection or read-only MCP lookups while running this workflow.

When an incident resolves or the user asks for a final pass, run retrospective cleanup before treating the page as final: refresh PagerDuty and Dynatrace status, normalize exact timestamps, remove stale live-state wording, separate onset evidence from late secondary events, and prune follow-ups to only the unresolved work that still matters.
