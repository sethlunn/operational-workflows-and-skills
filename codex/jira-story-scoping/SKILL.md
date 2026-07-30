---
name: jira-story-scoping
description: "Scope work into well-formed Jira stories from an interactive description, a Confluence design document, or a preloaded context file containing a story breakdown. Use when the user asks Codex to scope, break down, or draft Jira stories for planned work: gather requirements, decompose into right-sized stories, draft the Key Details fields (Description, Acceptance Criteria, Definition of Done) within strict size limits, flag edges where an engineer-provided code example would improve implementation accuracy, and create the stories in Jira after review. Do not use for implementing an existing story or for incident follow-up planning."
---

# Jira Story Scoping

Read [../../workflows/jira-story-scoping.md](../../workflows/jira-story-scoping.md) before taking action.

Follow the shared workflow:

- gather requirements interactively or read the complete design document or breakdown file
- decompose into independently deliverable, roughly one-PR stories with explicit dependency edges
- draft every story against [../../references/jira-story-writing-standard.md](../../references/jira-story-writing-standard.md) in the shape of [../../templates/scoped-jira-story.md](../../templates/scoped-jira-story.md)
- keep descriptions within the size limit, acceptance criteria to at most three scenarios, and Definition of Done to at most three story-specific bullets over the team standard
- flag `Engineer input needed` edges and request concrete artifacts such as a permalink to similar logic in another service
- create or edit Jira issues only after the user approves the drafts or explicitly pre-authorized creation, then verify by reading the issues back
