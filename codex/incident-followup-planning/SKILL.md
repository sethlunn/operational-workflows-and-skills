---
name: incident-followup-planning
description: "Validate an existing incident Confluence document and a separate analysis or RAC against PagerDuty, Dynatrace, Atlassian, and local code evidence, then turn the validated conclusions into actionable Jira follow-up stories under an incident epic using the IFU story-field conventions."
---

# Incident Follow-up Planning

Read [../../workflows/incident-followup-planning.md](../../workflows/incident-followup-planning.md) before starting.
Read [../../references/jira-incident-followup.md](../../references/jira-incident-followup.md) before creating or updating Jira work.
Read [../../templates/incident-followup-story.md](../../templates/incident-followup-story.md) when drafting or writing follow-up stories.
Read [../../references/incident-investigation-lessons-2026-03-27.md](../../references/incident-investigation-lessons-2026-03-27.md) when the page or RAC looks too certain about alert noise, impact, ownership, or onset.
Read [../../workflows/pagerduty-incident-analysis.md](../../workflows/pagerduty-incident-analysis.md) only when you need to refresh the incident from raw evidence instead of validating the supplied artifacts.

Follow the shared workflow, default to `draft mode` unless the user explicitly asks to create or update Jira, and draft the story set one by one for review unless the user explicitly asks for bulk creation.
When writing IFU stories, populate the dedicated `Acceptance Criteria`, `Definition of Done`, and `Initiative Description` fields, then mirror `Acceptance Criteria` and `Definition of Done` into `Description` if the current Jira layout hides those fields from normal issue view.
