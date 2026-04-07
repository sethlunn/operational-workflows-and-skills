# Dynatrace Evidence Interpretation

Use this explanation when the telemetry is ambiguous and you need help understanding what competing signals might mean before you finalize a conclusion.

Use the workflows when you need to run the investigation steps. Use this page when the question is how to interpret what the evidence is already showing.

## Onset vs Late Secondary Events

- Anchor the first bad interval with exact timestamps before interpreting later changes.
- Do not treat events near mitigation, recovery, or incident closure as onset evidence unless they line up exactly with the first confirmed degradation.
- If a late event is real but probably secondary, say so explicitly.

## Chronic Background Defects

- If a defect predates the incident by hours or days and remains steady through the incident window, classify it as chronic background risk or a likely secondary contributor.
- Keep chronic defects in the write-up when they matter for customer impact or follow-up work, but do not promote them to onset cause without a matching timing change.

## Low-Load and Quiet-Traffic Alerts

- Low load by itself is not proof of outage.
- When a low-load or traffic-drop alert opens, look for onset-matching service errors, backlog growth, retry storms, dead-letter growth, or downstream failures before claiming the system is broken.
- If traffic drops without matching service-fault evidence, say the result is consistent with genuinely low customer interaction, upstream producer slowdown, or another non-outage explanation.
- For workers and consumers, check both the producer side and the consumer side before concluding whether the system or the demand pattern changed.

## Caller-Side vs Callee-Side Mismatch

- When a caller shows heavy downstream failures but the downstream service does not show matching local failure, CPU pressure, or error evidence, classify this as a caller-vs-callee mismatch.
- In that mismatch case, consider:
  - network or service-mesh path issues
  - client timeout or retry behavior
  - draining or rollout behavior
  - routing, DNS, or proxy issues
- Treat those as hypotheses until direct evidence proves one of them.

## Telemetry Gaps

- If service-scoped logs, exceptions, or events return `0` rows, do not immediately treat that as proof of service silence.
- Distinguish among:
  - genuinely quiet service behavior
  - wrong entity or field scope
  - telemetry or ingestion gap
- Record the gap in the result and say how it weakens the conclusion.

## Custom Metric Semantics and Monitor Math

- Do not assume a metric name tells you how the signal is aggregated.
- Validate whether the underlying metric is emitted as a count, gauge, distribution, or some other value type before interpreting a threshold breach.
- Inspect the metric dimensions and the emitting code when the alert meaning is ambiguous. A monitor labeled as a rate or approval signal may actually be amount-weighted, duration-weighted, or otherwise value-weighted.
- Separate the alert label from the proven monitor behavior in the write-up.

## Environment-Scoped and Composite Ownership

- Environment-level problems can bundle signals from multiple services or teams into one alert surface.
- Keep the routed or owning service as the document owner unless evidence clearly proves another service is the main failing path.
- Compare the routed service slice with the dominant breached series across dimensions. If another service dominates, call out mixed ownership or mis-scoping explicitly.
- Treat alert routing mistakes as findings of their own instead of forcing them into a service-local outage narrative.

## Deployment Correlation vs Code Causation

- A rollout can be incident-relevant even when the deployed code is not the functional root cause.
- If the deployment lines up with the first bad interval but the diff does not touch the failing path, classify the result more precisely:
  - rollout correlated
  - code change likely unrelated
  - rollout or platform behavior may have amplified the issue
- Do not collapse those into a generic `deployment caused it` conclusion.

## Wording Rules

- Separate:
  - direct evidence
  - interpretation
  - unresolved ambiguity
- Prefer explicit wording such as `count-based` versus `amount-weighted` when the monitor semantics materially affect the conclusion.
- Prefer `most likely`, `consistent with`, or `supported by indirect evidence` when mesh, routing, or proxy failure is inferred rather than directly observed.
