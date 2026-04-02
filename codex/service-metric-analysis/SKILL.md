---
name: service-metric-analysis
description: "Analyze a repo service or component to identify emitted metrics from code, query production telemetry in Dynatrace, and create or update Confluence documentation. Use when the user wants metric breakdowns, trend analysis, telemetry semantics, caveats, or a Confluence page for a service such as credit-line-change or another repo-owned component."
---

# Service Metric Analysis

Read [../../workflows/service-metric-analysis.md](../../workflows/service-metric-analysis.md) before starting.
Read [../../templates/service-metric-analysis-page.md](../../templates/service-metric-analysis-page.md) when creating or updating the Confluence page body.
Read [../../references/dynatrace-query-patterns.md](../../references/dynatrace-query-patterns.md) when you need starter DQL shapes.
Read [../../references/dynatrace-evidence-interpretation.md](../../references/dynatrace-evidence-interpretation.md) when metric evidence is ambiguous, long-window rollups drift, or a customer-level question may not be answerable from metrics alone.

Default to production unless the user explicitly asks for another environment.
Use fixed calendar windows for long-range history instead of trusting one giant scalar rollup.
