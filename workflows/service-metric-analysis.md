# Service Metric Analysis

Inspect a repo service or component to identify emitted metrics from code, analyze the metric telemetry in Dynatrace, and create or update a Confluence page.

Read `../workflows/service-analysis-common.md` before starting.
Read `../templates/service-metric-analysis-page.md` when creating or updating the page body.
Read `../references/dynatrace-query-patterns.md` when you need starter DQL shapes.
Read `../references/dynatrace-evidence-interpretation.md` when metric evidence is ambiguous, when long-window rollups do not match fixed-window sums, or when the user asks a customer-level question that metrics may not support.
Read `../references/telemetry-measurability.md` when the question depends on metric dimensions, historical rollups, or distinct-entity claims.
Read `../references/subagent-usage.md` when deciding whether to split code discovery from telemetry analysis.
Read `../templates/analysis-child-result.md` when bounded child investigations are being used.

## Subagent Posture

- Optional, not default.
- Good split:
  - code-path and metric-semantics discovery
  - Dynatrace history, breakdowns, and rollups
- Keep the parent as the canonical writer for the Confluence page.
- Avoid splitting small or already-localized questions.
- When split, have each child return `../templates/analysis-child-result.md`.

## Workflow

1. Resolve the analysis-specific question and page target.
- Write down the exact question being answered:
  - metric inventory
  - bypass or outcome breakdown
  - daily or monthly trends
  - duration analysis
  - customer-level question that may or may not be supported
- If the user gives a Confluence page, update that page. Otherwise, search for an existing page with the expected title and update it if present.

2. Identify the metrics in code.
- Inspect the service entry path and follow the call path to the metric emitter.
- Prefer `rg` searches for likely emitters such as:
  - `StatsD`
  - `IStatsDPublisher`
  - `Meter`
  - `Counter`
  - `Histogram`
  - `Gauge`
  - `Telemetry`
  - `AddTelemetry`
  - `Prometheus`
  - metric name fragments
- Record:
  - exact metric names
  - exact dimensions or tags emitted
  - where timing starts and stops
  - what makes a result count as bypassed, failed, or changed
  - conditions where no metric is emitted at all
- Distinguish event-count telemetry from identity-preserving telemetry. If `customerId`, `orderId`, or another unique identifier is absent from the metric dimensions, say so explicitly.

3. Verify the metric surface in Dynatrace.
- Use `fetch metric.series` first to inspect the retained dimensions before doing rollups.
- Confirm the production filter dimensions actually exist on the metric, such as `env`, `service`, or similar retained tags.
- Prefer exact filters over fuzzy searches.
- If a metric is present in code but absent in Dynatrace, say so explicitly and narrow the likely reasons:
  - environment mismatch
  - metric not emitted in the searched window
  - dimension mismatch
  - ingestion or retention issue

4. Calculate the requested analysis.
- For long histories, prefer:
  - fixed calendar months for monthly reporting
  - fixed days for daily averages
- Do not rely on one giant scalar query when fixed-window sums disagree with it.
- Compute at least:
  - total event count in scope
  - average per day
  - monthly-equivalent average when the history is partial
  - requested dimension breakdowns such as `isbypassed`, `changetype`, `testcohort`, or other retained dimensions
- When the user asks for distinct-customer or distinct-entity questions:
  - first verify whether the metric retains a unique identifier dimension
  - if not, state clearly that the metric stream cannot answer the question
  - only pivot to logs, spans, or events if the user explicitly wants a broader non-metric analysis

5. Explain why the telemetry looks the way it does.
- Tie the observed dimensions and percentages back to the code path.
- Call out important distinctions such as:
  - bypassed metric result vs top-level kill switch with no metric
  - `changetype=None` vs true no-op vs bypass path
  - timing metric including model build or post-assessment work
  - partial dimension coverage in Dynatrace vs total raw event count
- Separate:
  - direct evidence from code
  - direct evidence from Dynatrace
  - interpretation or inference

6. Write or update the Confluence page.
- Mirror the structure in `../templates/service-metric-analysis-page.md`.
- Include exact absolute timestamps and exact filters used.
- Include the exact DQL used for the core rollups.
- State all important caveats in the page body, not only in the final chat response.

## Confluence Content Standard

Use concise, decision-oriented prose.

The page should usually include:

- executive summary
- exact scope and date range
- code-path explanation
- metric inventory with retained dimensions
- historical summary
- daily and monthly averages
- breakdowns by the requested dimensions
- explicit limitations
- exact DQL used

If the user asks for customer-level conclusions and the metric does not preserve identity, say this explicitly:

- this metric supports event counts, not distinct-customer counts

If fixed-window sums and one-shot long-range scalars disagree, say this explicitly:

- fixed-window sums are being treated as the source of truth for the document

## Output Rules

- Link the created or updated Confluence page in the final response.
- Distinguish:
  - event counts
  - unique-entity counts
  - what is measurable vs not measurable from the metric stream
- When a dimensioned breakdown does not sum to the full total, call out likely missing-dimension coverage instead of hiding the mismatch.
