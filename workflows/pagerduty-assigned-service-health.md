# PagerDuty Assigned Service Health

Discover the services currently assigned to the current PagerDuty user, map them to Dynatrace, and summarize health with exact dates.

Read `../workflows/service-analysis-common.md` before starting.
Read `../references/dynatrace-fast-path.md` when you start Dynatrace mapping or health queries.
Read `../references/slack-setup.md` when Slack publishing is requested or Slack is not configured.
Read `../references/subagent-usage.md` when deciding whether a large owned-service set is worth splitting.

## Subagent Posture

- Usually stay local.
- Consider subagents only when the owned service set is large enough that per-service health checks can run in parallel without overlapping scope.
- Keep the parent responsible for the final health rollup and any Slack publication.

## Workflow

1. Confirm tool availability and resolve the health window.
- Fail fast if PagerDuty cannot return the current user.
- If Dynatrace is unavailable, say so explicitly after PagerDuty discovery instead of inventing health data.
- If the user does not provide a time window, default to the last weekend in the user's timezone.
- A good default is previous Saturday `00:00` through Monday `00:00` in the user's timezone.

2. Resolve the current user's active PagerDuty ownership.
- Get the current PagerDuty user id first.
- Resolve active on-calls for that user.
- Treat an on-call entry as active when either:
  - both `start` and `end` are missing
  - the current timestamp falls inside the returned `start` and `end` window
- For each active escalation policy, resolve the policy and use its attached services as the authoritative ownership list.
- Deduplicate by PagerDuty service id and keep the service name, escalation policy name or id, and the on-call level that surfaced it.
- If the current user has no active matching on-calls, say that clearly and stop.

3. Map PagerDuty services to Dynatrace entities.
- Split PagerDuty services into buckets before touching Dynatrace:
  - repo-backed services that appear to match a local directory under `services/`
  - internal projections, workers, or other non-primary internals
  - external or platform services such as shared vendors or infrastructure
- For repo-backed services, prefer the fast path in `../references/dynatrace-fast-path.md`.
- Use local code or service naming conventions when needed to disambiguate repo-backed runtime services from workers, sidecars, or helpers.
- Keep prod and non-prod separate from the start.
- For repo-backed HTTP services, try the most specific prod-style names first:
  - `<service-name>-primary`
  - exact repo service name
  - a small number of obvious runtime variants only if the first two fail
- Prefer runtime application entities over queues, databases, sidecars, background helpers, or `Unexpected service` unless the user explicitly asks for those.
- If the mapping is ambiguous, narrow to plausible `SERVICE-*` runtime candidates before running metrics queries.
- If a PagerDuty service cannot be mapped confidently, mark it as unmapped or partial instead of guessing.

4. Assess health for each mapped prod service.
- Start with lightweight, scoped checks.
- Run a problem check before metrics so obvious problem signal can short-circuit deeper analysis.
- Batch problem and metric queries across confirmed prod `SERVICE-*` ids whenever practical.
- For request volume, failure rate, and latency, keep queries scoped only to the confirmed prod entity ids and group results by service id or service name.
- For notable endpoints and status codes, drill down only for services that already show elevated failures, latency, or active problems.
- Surface the highest-signal items: top failing endpoints, unusual status codes, or latency outliers.
- If multiple Dynatrace prod entities map to one PagerDuty service, aggregate the summary but list the contributing entity ids.

5. Report the result clearly.
- Lead with the exact assessed window and timezone.
- Summarize by PagerDuty service:
  - escalation policy
  - prod Dynatrace entity ids and names
  - open or recent problems
  - request volume
  - failure rate
  - latency
  - notable endpoints or status codes
  - confidence or mapping caveats
- Put non-prod mappings, unmapped services, and uncertain matches in separate sections.
- State when the result is partial because PagerDuty ownership was found but Dynatrace mapping was ambiguous or missing.

6. Publish to Slack when requested.
- Only do this when the user explicitly asks to post or publish the result.
- Fail fast with a clear message if Slack is unavailable or not configured.
- Prefer the default configured channel when none is specified.
- Post one parent summary message first, then one threaded reply per service.
- Keep the parent message short:
  - exact time window
  - total services checked
  - healthy, degraded, and unmapped counts
  - major caveats
- Keep each thread reply focused on one service:
  - service name
  - overall status
  - problems
  - request volume
  - failure rate
  - latency
  - notable endpoints or status codes
  - mapping confidence or caveats
- If Slack posting partially fails after the parent message is posted, report that partial failure and include the parent thread timestamp in the response.

## Output Rules

- Treat PagerDuty escalation-policy services as the source of truth for ownership.
- Clearly separate prod from non-prod in both analysis and summary.
- Do not print secrets.
