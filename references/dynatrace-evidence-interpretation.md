# Dynatrace Evidence Interpretation

Use this reference when the telemetry is ambiguous and you need to avoid over-claiming.

## Onset vs Late Secondary Events

- Anchor the first bad interval with exact timestamps before interpreting later changes.
- Do not treat events near mitigation, recovery, or incident closure as onset evidence unless they line up exactly with the first confirmed degradation.
- If a late event is real but probably secondary, say so explicitly.

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
- Prefer `most likely`, `consistent with`, or `supported by indirect evidence` when mesh, routing, or proxy failure is inferred rather than directly observed.
