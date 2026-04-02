# Incident Follow-up Planning

Validate an existing incident page and a second analysis, then turn the validated findings into actionable Jira follow-up work under an epic.

## Inputs

Accept any of:

- a Confluence incident page URL or page id
- a separate RAC, final assessment, or another agent's summary
- a Jira epic key or epic URL for the follow-up work
- optional PagerDuty incident ids, Dynatrace problem ids, or service names

Read `../references/incident-investigation-lessons-2026-03-27.md` before tightening claims that involve duplicate incidents, environment-scoped alerts, quiet-traffic noise, chronic background defects, or custom-metric semantics.
Read `../references/jira-incident-followup.md` before creating or updating Jira work.
Read `../references/subagent-usage.md` when deciding whether to split claim validation into independent evidence tracks.
Read `../templates/analysis-child-result.md` when bounded child investigations are being used.
Read `../templates/incident-followup-story.md` when drafting stories.
Read `../workflows/pagerduty-incident-analysis.md` only when the supplied document and summary are too weak and you need to refresh the incident from raw evidence.

## Subagent Posture

- Optional, not default.
- Use subagents only when there are multiple independent claims or evidence tracks worth validating in parallel.
- Keep final story synthesis, epic cleanup decisions, and Jira writing in the parent thread.
- When split, have each child return `../templates/analysis-child-result.md`.

## Execution Posture

Default to `draft mode` unless the user explicitly asks to create or update Jira issues or to rewrite the epic.

### Draft Mode

Use this when the user wants validation, an opinion, or story drafts but has not explicitly asked for Jira writes.

In `draft mode`:

- fetch and validate the supplied Confluence page and RAC
- use PagerDuty, Dynatrace, Atlassian, and local read-only inspection as needed
- draft the follow-up stories and recommend epic cleanup when useful
- do not create or update Jira issues

### Write Mode

Use this when the user explicitly asks to create or update the epic or stories.

In `write mode`:

- run the same validation as `draft mode`
- draft stories for review unless the user explicitly asks for bulk creation
- create or update Jira issues once the story wording is approved
- verify the created issues and description layout after each write

### Permission Rules

Do not interrupt for routine read-only work.

Execute these directly when needed:

- local read-only commands such as `rg`, `sed`, `ls`, `find`, `cat`, `git diff`, `git show`, and `git status`
- PagerDuty MCP reads
- Dynatrace MCP reads
- Atlassian read operations

Only interrupt when one of these is true:

- the action would create or update Jira, Confluence, PagerDuty, or Dynatrace configuration and the user did not ask for that
- the action is destructive or config-changing
- the action needs sandbox or network escalation that cannot be avoided

## Workflow

1. Resolve the source artifacts.
- Fetch the Confluence page, the external RAC or summary, and the Jira epic.
- Keep the exact page title, incident ids, problem ids, story keys, and the current date in the user's timezone.
- Capture the existing epic title and preexisting children before proposing cleanup.

2. Refresh current state before validating old wording.
- Re-check PagerDuty incident status, Dynatrace problem status, and any current Jira status that the write-up or summary references.
- Treat stale live-state phrases such as `ACTIVE`, `acknowledged`, or `still open` as point-in-time wording unless the current systems still match them.
- Normalize key dates and times into exact absolute timestamps in the user's timezone.

3. Break the supplied analysis into distinct claims.
- Separate claims about impact, ownership, volume, metric meaning, chronic defects, root cause, related incidents, and total counts.
- Evaluate each claim as one of:
  - validated
  - partially validated
  - not validated
  - not checked
- Prefer direct evidence over narration.

4. Validate the claims against source evidence.
- Use Confluence, PagerDuty, Dynatrace, and local code inspection as needed.
- Distinguish routed service ownership from dominant breached series ownership.
- Distinguish acute onset from chronic background defects.
- For low-load alerts, test quiet traffic versus service failure before supporting an outage story.
- For custom metrics, verify what the metric actually measures before accepting a plain-language summary.
- Do not claim `zero customer impact`, `false positive`, or `stale config` unless the evidence supports those exact words.

5. Produce the assessment.
- Start with findings, not a generic summary.
- State clearly what was independently validated, what was only partially supported, and what remains inference.
- Tighten overclaiming language. Prefer phrases such as `no confirmed customer impact` unless impact is directly evidenced.
- Keep the opinion short, concrete, and specific to the document and RAC being reviewed.

6. Convert the validated issues into workstreams.
- Group follow-up work by one actionable problem area per story.
- Separate monitor tuning, routing or deduplication, and service defects into different stories.
- Keep broad epics and narrow stories distinct.
- Leave unrelated preexisting child issues alone unless the user explicitly asks to repurpose, close, or unlink them.

7. Draft stories for review.
- Draft the story set one by one by default.
- Each draft should include:
  - title
  - description with only narrative, `Scope`, and `Out of Scope`
  - acceptance criteria
  - definition of done
  - initiative description
- Keep acceptance criteria testable and definition of done operational.

8. Discover Jira fields before writing.
- Use Jira project and issue-type metadata to confirm the field names and ids for the target project and issue type.
- For IFU stories, confirm the dedicated fields for `Acceptance Criteria`, `Definition of Done`, `Initiative Description`, and any `Slack Channel` field before creating or updating issues.
- Do not assume that fields visible in the API are visible in the current Jira issue layout.

9. Create or update Jira stories.
- Populate the dedicated fields first.
- If the current Jira layout hides `Acceptance Criteria` or `Definition of Done`, mirror those sections into `Description` after create or update so the issue is readable in the normal UI.
- Verify each created story by re-reading it and confirming epic linkage.
- Default to creating stories one by one after user approval unless the user explicitly asks for bulk creation.

10. Clean up the epic only when asked.
- If the user asks, retitle and rewrite the epic to reflect the validated follow-up initiative rather than the raw incident title.
- Keep the epic description aligned to the validated conclusion, not the first live-triage narrative.
- Do not rewrite or remove unrelated existing child issues unless the user explicitly asks.

11. Prepare the channel update only when asked.
- Keep Slack follow-up messages short and concrete.
- Mention the validated conclusion and the story keys created under the epic.
- Only mention manual monitoring changes if the user says they made them. Do not imply the agent changed production monitoring.

## Output Rules

- Findings first, then opinion, then story drafts or creation status.
- Cite exact issue keys, incident ids, problem ids, and absolute dates when available.
- Be explicit about what was independently validated versus inferred.
- Prefer scoped evidence over broad counts or strong adjectives.
- Keep story titles action-oriented and narrow enough to hand to one team.
- In `write mode`, verify the created stories before treating the task as complete.
