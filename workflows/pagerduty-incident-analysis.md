# PagerDuty Incident Analysis

Analyze a PagerDuty incident end to end: resolve the incident and owning service in PagerDuty, run a high-level Dynatrace investigation, create the parent Confluence page early, coordinate deeper bounded Dynatrace investigations, then synthesize the results into a timestamped incident write-up.

## Inputs

Accept any of:

- PagerDuty incident id such as `Q2YLVYYF9DVCJK`
- PagerDuty incident URL such as `https://zipcous.pagerduty.com/incidents/Q2YLVYYF9DVCJK`
- A request to investigate and document an incident in Confluence

Read `../workflows/dynatrace-investigation.md` before running the high-level sweep or any child investigation.
Read `../templates/incident-analysis-page.md` for the parent Confluence page shape.
Read `../templates/dynatrace-investigation-result.md` for the child-investigation return contract.

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

4. Run the high-level Dynatrace sweep first.
- Start with a broad but scoped incident investigation using the incident branch of `../workflows/dynatrace-investigation.md`.
- Use the alert surface, primary service, mapped entities, and incident window as the starting scope.
- Start with the highest-signal, lowest-cost checks:
  - active or recent problems
  - request or processing volume changes
  - failure rate
  - latency or duration
  - top failing endpoints or operations
  - relevant logs, exceptions, spans, or events
- Scope queries to the mapped entity ids and the incident window.
- Produce an initial answer to:
  - what looks broken
  - when it first became clearly broken
  - whether the blast radius appears isolated or broad
  - which deeper investigation tracks are justified
- Distinguish observed evidence from hypotheses. The purpose of this step is to narrow the queue, not to prove every possible root cause.

5. Create or update the parent Confluence page early.
- Search for an existing page for the same incident id before creating a new one.
- Update an existing page unless the user explicitly asks for a new page.
- Use the timestamp prefix rule from `../references/confluence-routing.md`.
- Create the page before deep child investigations so the incident has a canonical parent document from the start.
- Populate the initial page with:
  - incident summary and routing
  - exact investigation window
  - PagerDuty and Dynatrace entity mapping
  - initial high-level Dynatrace findings
  - the current investigation queue

6. Build the bounded child-investigation queue.
- Convert the high-level sweep into a small number of narrow tracks.
- Prefer a few high-signal tracks over many weak ones.
- Typical tracks include:
  - primary service failure path
  - one secondary impacted service
  - one downstream database, cache, or storage path
  - one third-party or internal dependency path
  - one identifier, trace, or propagation validation path
- Each track must have:
  - one exact question
  - one exact time window
  - one bounded entity or dependency scope
  - one reason it is worth investigating
- Do not spawn parallel tracks for every alert or every dependency. The queue should stay small enough that the parent can still synthesize it coherently.

7. Run child Dynatrace investigations.
- When child agents are available, use them for bounded tracks in parallel.
- When child agents are not available, run the same tracks sequentially while preserving the same scope boundaries and result contract.
- Each child should use `../workflows/dynatrace-investigation.md` as the evidence-gathering playbook for its assigned track.
- Child investigations should focus on one scope such as:
  - one service path
  - one dependency path
  - one data-validation chain
- Child investigations should not all write directly to the parent Confluence page.
- The parent incident workflow remains the canonical writer for the parent page.
- If the platform supports subpages or temporary scratch docs and they materially help, child investigations may write there, but the parent must still merge the final result into the main incident page.

8. Collect child results using a strict evidence contract.
- Each child must return a structured result package using `../templates/dynatrace-investigation-result.md`.
- Required fields:
  - exact question answered
  - exact time window searched
  - exact entities, dependencies, objects, or fields searched
  - exact queries that materially support the result
  - direct evidence
  - interpretation
  - confidence
  - strongest unresolved gap
  - next best narrow query if still unresolved
- Reject vague child conclusions that do not cite scope and evidence.

9. Update the parent page with child findings.
- Add one subsection per completed child investigation.
- Preserve which findings are direct evidence versus interpretation.
- Note unresolved or contradictory tracks explicitly instead of forcing a premature conclusion.
- Keep the parent page current as the investigations return so responders can follow progress.

10. Run a cross-investigation assessment.
- After the child tracks return, perform a synthesis pass in the parent workflow or a dedicated synthesis agent.
- The synthesis pass should answer:
  - which explanation is best supported across all tracks
  - which tracks support, weaken, or contradict each other
  - whether the incident is primarily service-local, downstream-driven, or platform-wide
  - where the strongest remaining blind spot sits
- The final assessment should stitch together the broad sweep and the child investigations, not replace them.

11. Finalize the Confluence page.
- Ensure the page contains the high-level sweep, the child investigations, the cross-investigation assessment, and the follow-ups.
- Leave exact timestamps, entity ids, and key queries in the page so the write-up is auditable.

## Output Rules

- Prefer PagerDuty incident and service metadata over guesswork.
- Prefer scoped Dynatrace evidence over broad scans.
- Use the high-level Dynatrace sweep to decide what deserves deeper work before spawning child investigations.
- Keep one canonical writer for the parent Confluence page.
- Use child investigations only for narrow, defensible scopes.
- Require every child investigation to return a structured evidence package.
- Synthesize child results into a cross-investigation assessment before finalizing the page.
- Use exact absolute dates and timestamps, not only relative phrases.
- Link the PagerDuty incident and the created or updated Confluence page.
- State clearly when the result is partial because mapping or evidence is ambiguous.
