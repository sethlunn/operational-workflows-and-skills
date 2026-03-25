# PagerDuty Incident Analysis

Analyze a PagerDuty incident end to end: resolve the incident and owning service in PagerDuty, investigate the likely cause and blast radius in Dynatrace, then publish a timestamped Confluence write-up.

## Inputs

Accept any of:

- PagerDuty incident id such as `Q2YLVYYF9DVCJK`
- PagerDuty incident URL such as `https://zipcous.pagerduty.com/incidents/Q2YLVYYF9DVCJK`
- A request to investigate and document an incident in Confluence

## Workflow

1. Resolve the incident in PagerDuty.
- Extract the incident id from the user's input before calling PagerDuty tools.
- Call `get_incident` first and keep:
  - incident id
  - title
  - urgency
  - status
  - created timestamp
  - resolved timestamp if present
  - assigned responders
  - primary `service.id`
- Gather more context only when needed:
  - `list_alerts_from_incident`
  - `list_incident_notes`
  - `list_incident_change_events`
  - `get_past_incidents`
  - `get_related_incidents`

2. Resolve service ownership and routing.
- Use the incident's primary `service.id` as the default ownership key.
- Call `get_service` for that service.
- Treat `service.escalation_policy.id` as the authoritative escalation-policy mapping.
- Call `get_escalation_policy` to get the human-facing escalation policy name and related services.
- Read `../references/confluence-routing.md` before choosing a Confluence destination.
- If the escalation policy is not mapped there, stop and ask the user where to save the page.
- If the incident clearly spans services owned by multiple mapped policies, ask the user which folder should own the write-up.

3. Build the investigation window.
- Default to:
  - start: `incident created time - 30 minutes`
  - end: `incident resolved time + 15 minutes`
- If the incident is still open, use the current timestamp as the end.
- Expand the window when notes, alerts, or change events indicate earlier degradation or slower recovery.
- Always convert the final window into exact absolute timestamps in the user's timezone.

4. Investigate in Dynatrace.
- Start with the highest-signal, lowest-cost checks:
  - active or recent problems
  - request volume changes
  - failure rate
  - latency
  - top failing endpoints or operations
  - relevant logs, exceptions, spans, or events
- Scope queries to the mapped entity ids and the incident window.
- Prefer narrow entity discovery and narrow DQL over broad environment scans.
- For HTTP services, drill down to endpoints only after identifying spikes in failure or latency.
- For worker, queue, or projection incidents, focus on exceptions, processing failures, backlog symptoms, and relevant logs or events instead of request metrics.
- Distinguish observed evidence from inference. Label hypotheses as hypotheses.

5. Create or update the Confluence page.
- Search for an existing page for the same incident id before creating a new one.
- Update an existing page unless the user explicitly asks for a new page.
- Use the timestamp prefix rule from `../references/confluence-routing.md`.
- Use `../templates/incident-analysis-page.md` as the default page structure.

## Output Rules

- Prefer PagerDuty incident and service metadata over guesswork.
- Prefer scoped Dynatrace evidence over broad scans.
- Use exact absolute dates and timestamps, not only relative phrases.
- Link the PagerDuty incident and the created or updated Confluence page.
- State clearly when the result is partial because mapping or evidence is ambiguous.
