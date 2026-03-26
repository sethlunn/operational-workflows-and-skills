# Dynatrace Investigation

Use Dynatrace as a primary evidence source for rollout checks, PagerDuty-backed investigations, service debugging, and identifier-based data validation.

Read `../references/dynatrace-query-patterns.md` when you need starter DQL shapes for metrics, logs, spans, events, or GUID tracing.
Read `../templates/dynatrace-investigation-result.md` when this workflow is being used as a bounded child investigation that must return a structured evidence package.

## Use Cases

Choose the workflow branch that matches the user's starting artifact.

### 1. Deployment Rollout Check

Use when the user wants to know whether a deployment, release, or rollout caused degradation.

Typical inputs:

- service name
- deployment or rollout time
- environment
- suspected symptom

### 2. PagerDuty-Backed or Incident Investigation

Use when the user wants to investigate a live or recent incident, especially one that starts from PagerDuty.

Typical inputs:

- PagerDuty incident id or URL
- service name
- incident time window
- symptom summary

### 3. Service Debugging

Use when the user wants to debug a service, endpoint, job, consumer, dependency failure, or user-facing error.

Typical inputs:

- service name
- endpoint, operation, consumer, or job name
- error text
- status code
- exception type
- customer, merchant, order, or request identifiers

### 4. GUID or Data Validation Trace

Use when the user wants to search for a GUID, request id, order id, customer id, merchant id, correlation id, BI event key, or similar value across telemetry.

Typical inputs:

- one or more exact identifiers
- an expected downstream event, warehouse record, or BI event
- a time window
- an optional owning service

## Shared Preflight

Run these steps before branching.

1. Normalize the investigation target.
- Convert relative dates into exact timestamps in the user's timezone.
- Normalize exact ids before searching:
  - original form
  - lowercase or uppercase variants when relevant
  - dashless GUID form only when the source system is known to emit it that way
- Write down the exact question being answered:
  - whether a rollout caused regression
  - what caused an incident
  - why a service failed
  - where an identifier trail stops

2. Resolve the narrowest useful entity scope.
- If the user gives a service name, start with `find_entity_by_name`.
- Prefer production application `SERVICE-*` entities over sidecars, projections, infra helpers, or `Unexpected service` unless the user explicitly wants those.
- When entity discovery is ambiguous, use narrow Smartscape or entity-scoped DQL to reduce the candidate set before running broader analysis.
- Keep prod and non-prod separate from the start.
- Record the exact entities examined so the final answer can state what was and was not searched.

3. Classify the runtime shape before choosing evidence.
- HTTP or API service
- async worker, consumer, projection, or queue processor
- scheduled job or batch process
- downstream dependency or shared platform issue
- telemetry or BI validation path
- This choice determines which signal should be checked first and which regressions are actually meaningful.

4. Choose the first evidence source based on the question.
- Rollout check:
  - `list_problems`
  - request or processing volume
  - failure rate
  - latency or duration
  - deployment or change events if available
- PagerDuty-backed or incident investigation:
  - `list_problems`
  - service-level failure, latency, and volume changes
  - top failing endpoints or operations
  - logs, exceptions, spans, or events
- Service debugging:
  - workload-specific coarse signal first
  - then logs, exceptions, spans, and endpoint or operation detail
- GUID or data validation trace:
  - discover the data object and field names first when unclear
  - then search logs, spans, events, and business telemetry fields in expected order

5. Keep queries narrow.
- Scope by exact entity ids whenever possible.
- Scope by a concrete time window before searching logs, spans, or events.
- Prefer exact-match filters over `contains` when the field is known.
- Use `generate_dql_from_natural_language` when it is safer or faster than hand-writing DQL.
- Use `fetch dt.semantic_dictionary.models` when the right data object or field is unclear before guessing at schema names.
- Prefer proving or disproving one concrete hypothesis per query over writing one broad query that mixes concerns.

## Shared Interpretation Rules

Apply these rules in every branch before declaring a root cause.

1. Separate onset from late secondary events.
- Anchor the first bad interval with exact timestamps before interpreting later changes.
- Do not treat events near mitigation, recovery, or incident closure as onset evidence unless they line up exactly with the first confirmed degradation.
- If a late event is real but probably secondary, say so explicitly.

2. Compare caller-side and callee-side evidence before blaming the callee.
- When a caller shows heavy downstream failures but the downstream service does not show matching local failure, CPU pressure, or error evidence, classify this as a caller-vs-callee mismatch.
- In that mismatch case, consider:
  - network or service-mesh path issues
  - client timeout or retry behavior
  - draining or rollout behavior
  - routing, DNS, or proxy issues
- Treat those as hypotheses until direct evidence proves one of them.

3. Handle telemetry gaps explicitly.
- If service-scoped logs, exceptions, or events return `0` rows, do not immediately treat that as proof of service silence.
- Distinguish among:
  - genuinely quiet service behavior
  - wrong entity or field scope
  - telemetry or ingestion gap
- Record the gap in the result and say how it weakens the conclusion.

4. Distinguish deployment correlation from code causation.
- A rollout can be incident-relevant even when the deployed code is not the functional root cause.
- If the deployment lines up with the first bad interval but the diff does not touch the failing path, classify the result more precisely:
  - rollout correlated
  - code change likely unrelated
  - rollout or platform behavior may have amplified the issue
- Do not collapse those into a generic "deployment caused it" conclusion.

## Branch Playbooks

### Deployment Rollout Check

Use this branch when the user is asking whether a release or deployment caused customer-impacting degradation.

1. Define the rollout window.
- Anchor the analysis on the exact rollout start time if known.
- Default to a short comparison window on both sides of the rollout:
  - pre-window: `rollout start - 30 minutes` to `rollout start`
  - post-window: `rollout start` to `rollout start + 30 minutes`
- Expand the windows when the rollout was gradual, traffic is sparse, or degradation appears delayed.

2. Confirm the comparison is valid.
- Check whether traffic or processing volume is reasonably comparable between the pre-window and post-window.
- If the service was idle, heavily bursty, or partially rolled out, say the comparison is weak before drawing conclusions.
- Separate normal demand shifts from deployment-correlated regressions.

3. Start with high-signal regression checks.
- Run `list_problems` in the rollout window.
- Check service-level request or processing volume, failure rate, and latency or duration.
- Check deployment or change events if the environment exposes them.
- Note whether the first visible degradation starts before, during, or after the rollout.

4. Drill into the workload-specific failure surface.
- For HTTP services:
  - identify the endpoints or operations driving the regression
  - compare status code mix, exception types, and dependency failures
- For workers or consumers:
  - compare processing throughput, failure counts, retry noise, and backlog or lag symptoms
- For scheduled jobs:
  - compare run starts, completions, duration, skipped runs, and partial completions

5. Classify the conclusion explicitly.
- `correlated regression`
- `no regression observed`
- `inconclusive because comparison quality is weak`
- `degradation started before rollout`
- `degradation appears driven by downstream dependency or shared platform`
- `rollout correlated but code change likely unrelated`

6. Support the conclusion with direct evidence.
- Include the exact windows compared.
- Include the exact entities checked.
- Include the metrics, problems, logs, or events that make the conclusion defensible.

### PagerDuty-Backed or Incident Investigation

Use this branch when the investigation starts from an incident or from an incident-like symptom.

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

### Service Debugging

Use this branch when the question is a direct debugging question rather than a formal incident.

1. Identify the failing workload shape.
- HTTP or API request path
- async worker, projection, or consumer path
- scheduled job or batch execution
- downstream dependency failure
- If the workload shape is unclear, use topology, logs, and operation names to classify it before deep searching.

2. Define the symptom precisely.
- Capture the exact failing endpoint, operation, exception, status code, or user-visible symptom.
- Anchor the first search on the narrowest known combination of:
  - entity
  - time window
  - operation name
  - exception type
  - identifier

3. Follow the workload-specific drill-down.
- HTTP or API:
  - check failing endpoints or operations
  - compare status classes and top exception types
  - use spans to identify the first failing downstream hop
- Worker, consumer, or projection:
  - check failure spikes, retry noise, poison or repeat processing patterns, and backlog or lag symptoms
  - use logs and spans to isolate the failing stage
- Scheduled job or batch:
  - check whether runs started, completed, took longer than normal, or stopped entirely
  - look for partial progress markers and terminal exceptions
- Downstream dependency:
  - look for one dependency signature repeated across multiple callers, endpoints, or operations
  - confirm whether the service is failing because of its own code or because a shared downstream is timing out, rejecting, or returning bad data

4. Stop when the first credible failing hop is identified.
- Prefer identifying the earliest confirmed failing boundary over collecting every downstream symptom.
- If the root cause remains unclear, say which hop is last known good and which hop is first known bad.

### GUID or Data Validation Trace

Use this branch when the user wants to prove that an identifier or BI event did or did not flow through the expected path.

1. Build the identifier set and transformation hypotheses.
- Record every exact identifier supplied by the user.
- Add only justified variants:
  - case variants
  - dashless GUID
  - derived correlation id or request id when there is evidence of that relationship
- Do not invent relaxed search variants without a reason.

2. Discover the telemetry objects and fields before concluding absence.
- If the expected event, record, or BI signal is not already known, use semantic dictionary or schema discovery first.
- Identify:
  - likely data object
  - exact field name
  - service or entity expected to emit it
- Do not conclude that an event is missing until the relevant object and field names have been validated.

3. Start from the earliest expected producer.
- Search the earliest service or event that should contain the identifier.
- Record the earliest confirmed observation with:
  - timestamp
  - entity
  - object type
  - field name
  - exact matched value

4. Walk the chain one hop at a time.
- Move from upstream producer to downstream consumer or BI emission.
- For each hop, record:
  - expected system
  - expected object
  - expected field
  - whether the identifier was found
  - the last confirmed timestamp
- Call out where the trail stops rather than jumping directly to the final missing artifact.

5. Be strict about “missing BI event” conclusions.
- Only claim a downstream or BI event is missing when:
  - upstream evidence shows the source event really happened
  - the searched downstream object and field are correct
  - the searched time window is wide enough for the expected delay
- Otherwise say the answer is partial and describe the strongest remaining uncertainty.

## Cross-Branch Heuristics

- Run `list_problems` before expensive drill-downs when the question smells like an incident or rollout regression.
- Avoid broad environment scans when topology or entity ids can answer the question.
- For HTTP services, do endpoint drill-down only after confirming overall regression or narrowing to one failing path.
- For workers, consumers, or projections, prioritize exceptions, backlog symptoms, retries, processing failures, and logs over request-rate analysis.
- For GUID tracing, search exact identifiers first and only relax the search if exact filters miss and there is evidence the identifier may be transformed.
- For BI-event validation, identify the actual telemetry object and field names before concluding an event is missing.
- Distinguish direct evidence from inference in every branch.

## Child-Investigation Contract

Use this section when another workflow, such as PagerDuty incident analysis, invokes this workflow as a bounded sub-investigation.

1. Take one narrow question.
- Accept one exact question, not a broad incident brief.
- Good examples:
  - whether one service path failed because of one downstream dependency
  - whether one database call family regressed in the incident window
  - where one identifier trail stopped

2. Keep the scope explicit.
- Record the exact time window.
- Record the exact entities, dependencies, objects, and fields searched.
- Do not silently widen from one scope to another.

3. Gather evidence using the matching branch.
- Use the incident branch for a narrow service-path or dependency-path investigation.
- Use the debugging branch for a specific failure path.
- Use the GUID or data-validation branch for propagation and missing-event questions.

4. Return a structured result package.
- Use `../templates/dynatrace-investigation-result.md`.
- Include:
  - exact question answered
  - exact time window searched
  - exact scoped entities and objects
  - direct evidence
  - interpretation
  - confidence
  - strongest unresolved gap
  - next best narrow query if unresolved

5. Do not act like the parent incident orchestrator.
- Do not try to explain the whole incident from one narrow track.
- Do not overwrite a shared parent document unless explicitly assigned sole ownership of a subpage or scratch artifact.
- Optimize for defensible evidence, not narrative completeness.

## Output Rules

- Prefer exact timestamps, entity ids, and query snippets over vague summaries.
- Lead with:
  - exact question answered
  - exact time window searched
  - exact entities and objects searched
  - primary conclusion
- Separate:
  - direct evidence
  - interpretation or inference
  - blind spots or unresolved ambiguity
- Include the exact DQL or key filtered query snippets that materially support the conclusion.
- Say explicitly when no evidence was found in the searched window and scope.
- Do not claim end-to-end absence unless the search scope and data objects actually support that conclusion.
- If unresolved, state the next best narrow query instead of recommending a broad re-scan.
