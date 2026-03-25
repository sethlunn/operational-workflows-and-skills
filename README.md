# Operational Workflows and Skills

Portable, company-specific operational workflows and reusable skill adapters for LLM agents.

This repo keeps the core logic in shared Markdown so multiple agent systems can reuse the same process with thin model-specific adapters.

## Structure

- `workflows/`
  - Shared operational procedures written to be readable by any model.
- `references/`
  - Stable routing, query-pattern, naming, and environment-specific reference material.
- `templates/`
  - Reusable output templates for Confluence pages and similar artifacts.
- `codex/`
  - Thin Codex adapters that wrap the shared workflow docs as `SKILL.md` skills.

## Current Workflows

- `babysit-pr`
  - Review a pull request, triage review threads, decide which comments are valid, and draft or post signed replies.
- `pagerduty-incident-analysis`
  - Resolve a PagerDuty incident, investigate it in Dynatrace, and publish a Confluence write-up.
- `dynatrace-investigation`
  - Investigate rollout health, incidents, service bugs, and GUID-based data validation in Dynatrace.
- `pagerduty-assigned-service-health`
  - Discover the current user's PagerDuty-owned services, map them to Dynatrace production entities, and summarize health for an exact time window.
- `service-endpoint-traffic-analysis`
  - Inventory a service's HTTP endpoints from code, map those endpoints to Dynatrace traffic, and publish Confluence guidance on usage tiers and cleanup candidates.

## Portability Rules

- Keep business logic in `workflows/`, `references/`, and `templates/`.
- Keep model-specific files as thin wrappers only.
- Avoid embedding secrets, tokens, or environment credentials in the repo.
- Prefer exact dates, entity ids, and query snippets over vague summaries.

## Codex Usage

To use a skill with Codex, copy or symlink one of the folders under `codex/` into `~/.codex/skills/`.

## Next Adapters

This layout is meant to support additional adapters later, for example:

- `claude/`
- `chatgpt/`
- `cursor/`
