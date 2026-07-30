# Scoped Jira Story Draft

Follow [../references/jira-story-writing-standard.md](../references/jira-story-writing-standard.md) for field ids, component rules, and size limits.

## Title

`<verb-first, specific, one line>`

## Description

`<optional trace line: Workstream · Breakdown ref · Effort>`

**Goal:** `<outcome this story delivers and why it matters>`

**Implementation guidance:** `<approach, pattern, or files to mirror, labeled as guidance>` (optional)

**Depends on:** `<blocking story keys, external prerequisites, or none>` (optional)

**Context:** `<deep links to the defining design sections, ADRs, or code>` (optional)

**Engineer input needed:** `<gap plus the exact artifact requested, such as a permalink to similar logic in another service>` (optional)

## Acceptance Criteria

- Given `<state>`, when `<condition applied to state>`, then `<observable outcome>`
- `<at most three scenarios; testable declarative bullets for non-behavioral stories>`

## Definition of Done

- Standard team Definition of Done satisfied `<link when published>`
- `<story-specific completion requirement>` (optional)
- `<story-specific completion requirement or exception>` (optional)

## Jira Write Notes

- Write `Description` to the system description field, `Acceptance Criteria` to `customfield_10053`, and `Definition of Done` to `customfield_10281`.
- Replace untouched template stubs in the dedicated fields instead of appending below them.
- Set the epic as parent, apply the agreed labels, and create `Blocks` links for every `Depends on` entry.
