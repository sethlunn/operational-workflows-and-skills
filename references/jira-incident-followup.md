# Jira Incident Follow-up

Use this note when creating or updating incident follow-up stories in Jira.

## Current IFU Field Map

These field ids were verified in the Quadpay Atlassian instance on `2026-04-01`. Reconfirm with Jira metadata if a create or update fails or if the issue type differs.

- `Acceptance Criteria`: `customfield_10053`
- `Definition of Done`: `customfield_10281`
- `Initiative Description`: `customfield_10446`
- `Slack Channel`: `customfield_11159`

## Story Layout

Use this content split by default:

- `Description`
  - narrative
  - `Scope`
  - `Out of Scope`
- `Acceptance Criteria`
  - dedicated Jira field
- `Definition of Done`
  - dedicated Jira field
- `Initiative Description`
  - dedicated Jira field

## Visibility Workaround

The IFU story layout may hide `Acceptance Criteria` and `Definition of Done` from the normal issue view even when those fields are populated in the backend.

When that happens:

- keep the dedicated fields populated
- mirror `Acceptance Criteria` and `Definition of Done` into `Description`
- verify the final issue by reading it back from Jira

## Verification Notes

- Confirm epic linkage after creation.
- If a user says the fields are missing, check the backend issue payload before assuming the write failed.
- As of `2026-04-01`, IFU stories did not require Details-field completion to move from `To Do` to `In Progress`, but re-check transitions if the workflow changes.
