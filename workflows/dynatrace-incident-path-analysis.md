# Dynatrace Incident Path Analysis

Read `../workflows/dynatrace-investigation.md` first, then use this playbook when the investigation starts from an incident or from an incident-like symptom.

This is the default Dynatrace branch for the high-level sweep inside `pagerduty-incident-analysis`, and for bounded child tracks that focus on one service path or dependency path.

## Workflow

1. If the user starts from PagerDuty, resolve the incident package first.
- Accept the resolved incident package from the PagerDuty workflow or build it directly.
- Keep:
  - incident id
  - primary `service.id`
  - primary service name
  - escalation policy id and name
  - exact investigation window
  - incident status and urgency
  - any secondary service clues from alerts, notes, or change events
- Treat this package as the default ownership and scoping contract for Dynatrace work.

2. Start coarse before drilling down.
- Run `list_problems` across the incident window.
- Check service-level request or processing volume, failure rate, and latency or duration.
- Identify the first clearly bad interval inside the incident window.

3. Establish blast radius before root-cause guessing.
- Determine whether the issue is isolated to:
  - one endpoint or operation
  - one consumer or worker path
  - one dependency
  - one tenant, merchant, or customer slice
  - a broader service or platform issue
- If secondary services appear involved, keep the primary incident service as the routing key unless evidence clearly points elsewhere.

4. Drill into the likely failing path.
- For HTTP services, move from service metrics to failing endpoints, then to exceptions, logs, and traces.
- For workers or projections, move from throughput and failure symptoms to exceptions, retries, logs, and relevant events.
- For dependency-heavy failures, identify the downstream that appears across the failing traffic before searching broadly.

5. Correlate with change context.
- Check whether deployments, config changes, or change events align with the first bad interval.
- Distinguish correlation from causation. If the change is only adjacent in time, say so.
- If the evidence points to caller-side failure without matching callee-side degradation, say whether the incident looks:
  - service-local
  - downstream-driven
  - network or service-mesh mediated
  - inconclusive because the telemetry is missing

6. Produce an incident-ready conclusion.
- Lead with exact incident window, primary service, and primary conclusion.
- Separate direct evidence from inference.
- State clearly when the result is partial because service mapping, telemetry coverage, or incident scope is ambiguous.
- When this branch feeds a Confluence write-up, return the evidence in a form that can drop directly into the incident analysis template.
