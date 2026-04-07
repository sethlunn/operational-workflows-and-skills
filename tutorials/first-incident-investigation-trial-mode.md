# Tutorial

- Title: First incident investigation in trial mode
- Audience: Operators or contributors who are new to this repo's incident flow
- Goal: Run one complete PagerDuty and Dynatrace incident investigation without publishing anything
- Starting knowledge assumed: Basic familiarity with Codex, PagerDuty, and Dynatrace
- Expected finish state: You have a parent-page-ready incident result and understand the normal trial-mode loop
- Safety or environment notes: This tutorial stays in `trial mode` and should not create or update external artifacts

# Prerequisites

- Required tools or access:
  - Codex CLI
  - access to the PagerDuty and Dynatrace tools used by the repo
  - a local service workspace you want Codex to use during the session
- Starting workspace or data:
  - the `operational-workflows-and-skills` repo
  - one incident id or incident URL you are allowed to inspect
- Estimated time:
  - about 10 to 20 minutes for a first run
- What to do if a prerequisite is missing:
  - if the launcher is not on your `PATH`, run it as `./scripts/codex-incident-session`
  - if the target workspace is not your current directory, pass it explicitly as the second argument
  - if tool access is missing, stop and fix access before starting the tutorial

# Guided Path

- Step 1:
  - Choose the service workspace Codex should use for the incident session.
  - If you are already in that workspace, run:
    - `codex-incident-session trial`
  - If you are launching from this repo or another directory, run:
    - `./scripts/codex-incident-session trial /path/to/services`
- What success looks like:
  - Codex starts with a preflight prompt for a PagerDuty and Dynatrace incident-investigation session in `trial mode`

- Step 2:
  - Let the preflight checks run.
  - If approval prompts appear for read-only tool access, approve them for the session.
  - Wait until Codex says preflight is complete and asks for the incident id.
- What success looks like:
  - the session reports that preflight is complete
  - no investigation has started yet
  - no Confluence page has been created or updated

- Step 3:
  - Provide one exact incident target.
  - Use either an incident id or a PagerDuty incident URL.
  - A safe prompt shape is:
    - `Use $pagerduty-incident-analysis to investigate PagerDuty incident <INCIDENT_ID> in trial mode. Do not publish anything.`
- What success looks like:
  - Codex resolves the PagerDuty incident
  - Codex builds the exact investigation window
  - Codex runs the high-level Dynatrace sweep and any bounded child investigations that are justified

- Step 4:
  - Wait for the final trial-mode result.
  - Read the returned summary, timeline, root-cause analysis, impact analysis, and evidence sections.
  - Confirm that the result is parent-page-ready rather than published.
- What success looks like:
  - you receive a structured incident result that could be published later
  - the result includes exact timestamps, scope, and evidence
  - no Confluence page link is created unless you explicitly asked for `publish mode`

# Checkpoint

- What the reader should now have:
  - one complete `trial mode` incident investigation result
  - a repeatable way to start future incident sessions
- What the reader should now understand:
  - `trial mode` is the default operating posture
  - the parent incident workflow is the canonical writer
  - child investigations are bounded evidence-gathering tracks, not separate incident narratives

# Recovery

- If a step did not match expectations:
  - if the launcher says it cannot find the workspace, rerun with an explicit second argument or set `CODEX_INCIDENT_WORKDIR`
  - if the session starts investigating before preflight finishes, stop and relaunch so the approval flow stays clean
  - if the agent starts to create or update a Confluence page, restate that the session must remain in `trial mode`
- Safe retry point:
  - restart from Step 1 with the same incident id
- Where to look next:
  - `workflows/pagerduty-incident-analysis.md`
  - `templates/incident-analysis-page.md`
  - `templates/dynatrace-investigation-result.md`

# Next Steps

- Related how-to guides:
  - `workflows/pagerduty-incident-analysis.md`
- Related reference:
  - `references/diataxis-writing-rules.md`
  - `references/confluence-routing.md`
- Related explanation:
  - `explanations/repo-architecture.md`
