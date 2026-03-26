---
name: dynatrace-investigation
description: "Investigate a deployment rollout, PagerDuty-backed incident, service bug, telemetry gap, or GUID-based data validation question in Dynatrace. Use when the user wants rollout health checks, debugging, incident investigation, or tracing exact ids such as GUIDs, order ids, customer ids, or correlation ids through logs, spans, events, or business telemetry."
---

# Dynatrace Investigation

Read [../../workflows/dynatrace-investigation.md](../../workflows/dynatrace-investigation.md) before starting.
Read [../../references/dynatrace-query-patterns.md](../../references/dynatrace-query-patterns.md) when you need starter DQL shapes or a trace-friendly query pattern.
Read [../../templates/incident-analysis-page.md](../../templates/incident-analysis-page.md) when the user wants an incident-style write-up.
Read [../../templates/dynatrace-investigation-result.md](../../templates/dynatrace-investigation-result.md) when this is a bounded child investigation feeding a parent incident workflow.
Read [../../references/confluence-routing.md](../../references/confluence-routing.md) only when the user wants the result published or routed to Confluence by owning team.

Choose the shared workflow branch that matches the user's starting artifact:

- deployment rollout
- incident
- debugging
- GUID or data validation trace

When invoked as a child investigation, stay within the assigned scope and return a structured evidence package instead of trying to narrate the entire incident.

When the evidence shows caller-side failure without matching callee-side degradation, or when service-local logs are missing, treat network or service-mesh explanations and telemetry-gap conclusions as hypotheses until the supporting evidence is explicit. Distinguish onset evidence from late secondary events.
