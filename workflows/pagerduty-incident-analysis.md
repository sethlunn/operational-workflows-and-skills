# PagerDuty Incident Analysis

Analyze a PagerDuty incident end to end: resolve the incident and owning service in PagerDuty, run a high-level Dynatrace investigation, create the parent investigation surface early, coordinate deeper bounded Dynatrace investigations, then synthesize the results into a publish-ready or published incident write-up.

## Inputs

Accept any of:

- PagerDuty incident id such as `Q2YLVYYF9DVCJK`
- PagerDuty incident URL such as `https://zipcous.pagerduty.com/incidents/Q2YLVYYF9DVCJK`
- A request to investigate and document an incident in Confluence

Read `../workflows/dynatrace-investigation.md` before running the high-level sweep or any child investigation.
Read `../workflows/dynatrace-incident-path-analysis.md` before the high-level incident sweep.
Read `../templates/incident-analysis-page.md` for the parent Confluence page shape.
Read `../templates/dynatrace-investigation-result.md` for the child-investigation return contract.
Read `../references/incident-investigation-lessons-2026-03-27.md` only when you want the concrete incident examples that motivated the current guardrails.

## Execution Posture

Default to `trial mode` unless the user explicitly asks to create, update, or publish a Confluence page.

### Trial Mode

Use this when the user wants the incident investigated but has not explicitly asked for publishing.

In `trial mode`:

- investigate the incident fully
- use PagerDuty, Dynatrace, and Atlassian read operations as needed
- use local read-only shell commands as needed
- do not create or update the Confluence page
- return a parent-page-ready result that can be published later

### Publish Mode

Use this when the user explicitly asks to create, update, or publish the Confluence document.

In `publish mode`:

- run the same investigation as `trial mode`
- create or update the parent Confluence page early
- keep that page current as child investigations return

### Permission Rules

Do not interrupt the investigation to ask permission for routine read-only work.

Execute these directly when needed:

- local read-only commands such as `rg`, `sed`, `ls`, `find`, `cat`, `git diff`, `git show`, and `git status`
- PagerDuty MCP reads
- Dynatrace MCP reads
- Atlassian read operations

Only interrupt when one of these is true:

- the action would create, update, or publish an external artifact and the user did not ask for that
- the action is destructive or config-changing
- the action needs sandbox or network escalation that cannot be avoided

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
- If the alert title, notes, or linked vendor surface includes an external problem id such as a Dynatrace `P-...`, check whether PagerDuty opened concurrent incidents on the same service for that same underlying problem.
- Treat those concurrent incidents as related or duplicate alert surfaces unless the evidence shows clearly separate failing paths.

2. Resolve service ownership and routing.
- Use the incident's primary `service.id` as the default ownership key.
- Call `get_service` for that service.
- Treat `service.escalation_policy.id` as the authoritative escalation-policy mapping.
- Call `get_escalation_policy` to get the human-facing escalation policy name and related services.
- Read `../references/confluence-routing.md` before choosing a Confluence destination.
- If the trigger is environment-scoped or composite but PagerDuty routed it to one primary service, keep that primary service as the default document owner unless evidence clearly proves the main failing path belongs elsewhere.
- Record mixed ownership explicitly in the page instead of re-homing the write-up mid-investigation just because one alert bundles multiple services.
- If the escalation policy is not mapped there, stop and ask the user where to save the page.
- If the incident clearly spans services owned by multiple mapped policies, ask the user which folder should own the write-up.

3. Build the investigation window.
- Default to:
  - start: `incident created time - 30 minutes`
  - end: `incident resolved time + 15 minutes`
- If the incident is still open, use the current timestamp as the end.
- Expand the window when notes, alerts, or change events indicate earlier degradation or slower recovery.
- Always convert the final window into exact absolute timestamps in the user's timezone.

4. Choose the operating mode.
- If the user explicitly asks to create, update, or publish the Confluence document, use `publish mode`.
- Otherwise use `trial mode`.
- State the chosen mode in the investigation notes or parent-page-ready output.

5. Run the high-level Dynatrace sweep first.
- Start with a broad but scoped incident investigation using `../workflows/dynatrace-incident-path-analysis.md` after the shared router and preflight in `../workflows/dynatrace-investigation.md`.
- Use the alert surface, primary service, mapped entities, and incident window as the starting scope.
- Classify the alert surface before root-cause guessing:
  - service-local problem
  - environment-scoped or composite problem
  - low-load or traffic-drop alert
  - custom metric threshold alert
- Start with the highest-signal, lowest-cost checks:
  - active or recent problems
  - request or processing volume changes
  - failure rate
  - latency or duration
  - top failing endpoints or operations
  - relevant logs, exceptions, spans, or events
- For low-load alerts, explicitly test quiet traffic versus service failure before writing an outage story.
- For custom metric alerts, validate the metric dimensions, dominant series, and emission semantics when the monitor meaning is not already clear.
- Scope queries to the mapped entity ids and the incident window.
- Produce an initial answer to:
  - what looks broken
  - when it first became clearly broken
  - whether the blast radius appears isolated or broad
  - which deeper investigation tracks are justified
- Distinguish observed evidence from hypotheses. The purpose of this step is to narrow the queue, not to prove every possible root cause.

6. Create the parent investigation surface.
- In `publish mode`:
  - search for an existing page for the same incident id before creating a new one
  - update an existing page unless the user explicitly asks for a new page
  - use the timestamp prefix rule from `../references/confluence-routing.md`
  - create the page before deep child investigations so the incident has a canonical parent document from the start
- In `trial mode`:
  - do not create or update the Confluence page
  - create an in-memory parent-page-ready outline using `../templates/incident-analysis-page.md`
- In both modes, populate the parent investigation surface with:
  - incident summary and routing
  - alert scope and ownership notes
  - related or duplicate incident notes
  - exact investigation window
  - PagerDuty and Dynatrace entity mapping
  - initial high-level Dynatrace findings
  - the current investigation queue

7. Build the bounded child-investigation queue.
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

8. Run child Dynatrace investigations.
- When child agents are available, use them for bounded tracks in parallel.
- When child agents are not available, run the same tracks sequentially while preserving the same scope boundaries and result contract.
- Each child should use `../workflows/dynatrace-investigation.md` as the router, then read the branch playbook that matches the assigned track.
- Child investigations should focus on one scope such as:
  - one service path
  - one dependency path
  - one data-validation chain
- Typical branch mapping:
  - service path or dependency path: `../workflows/dynatrace-incident-path-analysis.md`
  - specific failure-path debugging: `../workflows/dynatrace-service-debugging.md`
  - propagation or missing-event questions: `../workflows/dynatrace-guid-trace.md`
  - rollout-correlation questions: `../workflows/dynatrace-rollout-check.md`
- Child investigations should not all write directly to the parent Confluence page.
- The parent incident workflow remains the canonical writer for the parent page.
- If the platform supports subpages or temporary scratch docs and they materially help, child investigations may write there, but the parent must still merge the final result into the main incident page.

9. Collect child results using a strict evidence contract.
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

10. Update the parent investigation surface with child findings.
- Add one subsection per completed child investigation.
- Preserve which findings are direct evidence versus interpretation.
- Note unresolved or contradictory tracks explicitly instead of forcing a premature conclusion.
- In `publish mode`, keep the parent Confluence page current as the investigations return.
- In `trial mode`, keep the in-memory parent-page-ready result current as the investigations return.

11. Run a cross-investigation assessment.
- After the child tracks return, perform a synthesis pass in the parent workflow or a dedicated synthesis agent.
- The synthesis pass should answer:
  - which explanation is best supported across all tracks
  - which tracks support, weaken, or contradict each other
  - whether the incident is primarily service-local, downstream-driven, or platform-wide
  - whether the page is documenting a real service outage, a mixed-ownership alert, or alert-surface noise around a weaker service-local symptom
  - where the strongest remaining blind spot sits
- The final assessment should stitch together the broad sweep and the child investigations, not replace them.

12. Run retrospective cleanup when the incident is resolved or when the user asks for a final pass.
- Refresh the PagerDuty incident status and timestamps before finalizing the page.
- Refresh the Dynatrace problem status and timestamps before finalizing the page.
- Normalize every key time reference into exact absolute timestamps and correct mixed UTC or local-time wording.
- Rewrite the top summary in final tense once the incident is resolved.
- Remove or rewrite stale live-state phrases such as:
  - `ACTIVE`
  - `acknowledged`
  - `still resolving`
  - `best read so far`
- Separate onset evidence from late secondary or mitigation-adjacent events.
- If the page keeps live snapshots for auditability, label them as historical point-in-time notes rather than current status.
- Re-check the final conclusion after status normalization:
  - deployment correlated does not necessarily mean code-caused
  - downstream hot service does not necessarily mean downstream code-caused
  - late workload churn does not necessarily explain onset

13. Finalize the result.
- In `publish mode`:
  - ensure the page contains the high-level sweep, the child investigations, the cross-investigation assessment, the final status, and the follow-ups that still matter
  - leave exact timestamps, entity ids, and key queries in the page so the write-up is auditable
- In `trial mode`:
  - return the same parent-page-ready content without publishing it
  - if useful, tell the user that the result is ready to publish without requiring the investigation to be rerun

## Output Rules

- Prefer PagerDuty incident and service metadata over guesswork.
- Prefer scoped Dynatrace evidence over broad scans.
- Default to `trial mode` unless the user explicitly asks to publish.
- Use the high-level Dynatrace sweep to decide what deserves deeper work before spawning child investigations.
- Keep one canonical writer for the parent Confluence page.
- Use child investigations only for narrow, defensible scopes.
- Require every child investigation to return a structured evidence package.
- Synthesize child results into a cross-investigation assessment before finalizing the page.
- When the incident is resolved or the user asks for a final pass, run a retrospective cleanup pass before treating the write-up as final.
- Do not leave stale live-state wording at the top of a resolved incident page unless it is clearly labeled as a historical snapshot.
- Prune follow-ups so the final page keeps only unresolved, still-valuable work rather than every question that appeared during live triage.
- Do not ask permission for routine local read-only inspection or read-only MCP lookups.
- Use exact absolute dates and timestamps, not only relative phrases.
- In `publish mode`, link the PagerDuty incident and the created or updated Confluence page.
- In `trial mode`, return a publish-ready result without creating or updating the page.
- State clearly when the result is partial because mapping or evidence is ambiguous.
- Call out duplicate PagerDuty incidents, composite environment alerts, and alert-surface or routing noise explicitly when they are better supported than a clean service-local outage story.
