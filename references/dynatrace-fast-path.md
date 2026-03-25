# Dynatrace Fast Path

Use this sequence to reduce noisy Dynatrace discovery and health calls.

## Candidate Triage

1. Check whether the PagerDuty service looks repo-backed.
- Prefer local evidence first: a matching directory under `services/`, an obvious repo-style hyphenated name, or a known internal service name.
- Treat vendors and shared platforms separately. Do not spend the same mapping effort on external services unless the user explicitly wants them.

2. For repo-backed services, try prod-style names before generic names.
- Start with `<service-name>-primary`.
- Then try the exact repo service name.
- Only then try a small number of close variants.

## Cheap-First Resolution

1. Use the most specific candidate first.
2. Keep only plausible runtime `SERVICE-*` results for health analysis.
3. Ignore clusters, sidecars, background-thread services, queues, and `PROCESS-*` entities unless they are needed to disambiguate runtime ownership.
4. If the first pass still mixes prod and non-prod or runtime and projections, use narrow topology or narrowly scoped queries only against the surviving candidates.

## Query Batching

- After confirming prod `SERVICE-*` ids, batch them into as few calls as practical.
- Prefer one problem filter across the confirmed service ids instead of one call per service.
- Prefer one grouped metrics query over separate request-volume, latency, or failure-rate queries for each PagerDuty service.
- Only run endpoint or status-code drilldowns for services that already show active problems or unhealthy top-level metrics.

## Skip Rules

- Skip expensive mapping for external or platform services unless the user asked for them.
- Skip endpoint drilldowns when top-level metrics are clean.
- Mark ambiguous services as partial or unmapped instead of widening the search blindly.
