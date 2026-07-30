# Jira Story Writing Standard

Use this note when drafting or writing scoped Jira stories. It defines the Key Details field map, the content contract for each field, and the size limits that keep stories usable by implementing agents and by engineers reading Jira directly.

## Key Details Field Map

These field ids were verified against `DQ-9372` (project `DQ`, issue type `Story`) on `2026-07-30`. Reconfirm with Jira field metadata when a create or update fails or when the project or issue type differs.

- `Description`: the system `description` field
- `Acceptance Criteria`: `customfield_10053`
- `Definition of Done`: `customfield_10281`

Each Key Details field carries only its own content. Never place acceptance criteria or Definition of Done content inside the `Description` field; they always go in their dedicated fields, even when an older story such as `DQ-9372` embedded them in the description. Replace untouched template stubs in the dedicated fields instead of appending below them.

Do not populate the legacy template fields `Story Description` (`customfield_10108`) and `Purchasing Issue Outline` (`customfield_10123`). `Initiative Description` (`customfield_10446`) belongs to the incident follow-up flow in [jira-incident-followup.md](./jira-incident-followup.md), not to general scoping.

## Write Mechanics

Verified against the Atlassian MCP connector on `2026-07-30` while editing the DQ-9365 story set:

- The system `description` field accepts a Markdown string when the edit call sets `contentFormat: "markdown"`.
- The `customfield_10053` and `customfield_10281` doc fields reject strings with `Operation value must be an Atlassian Document`; pass an ADF object (`bulletList` of `listItem` > `paragraph`, inline code via text `marks`) for each in the same edit call.
- Bold-labeled description components and Confluence deep links survive the Markdown conversion; write them as normal Markdown.

## Dual-Audience Rule

Every story serves two readers at once:

- an implementing agent that needs unambiguous scope, dependencies, and exact references
- an engineer skimming Jira who will dismiss a wall of generated text

Meet both by being selective rather than compressed: include only context that changes what the implementer does, write it in plain sentences, and stop at the size limits. Never pad a section to look complete.

## Description Contract

Size limit: at most two short paragraphs, roughly 8 to 10 lines or sentences as a hard maximum, ideally 5 to 6.

Build the description from these bold-labeled components, in the `**Goal:** ...` format established by the DQ scoped-story style. This component list is exhaustive: acceptance criteria and Definition of Done never appear in the description. Include a component only when real context exists for it; `Goal` is always required.

- `Goal:` the outcome this story delivers and why it matters, in one or two sentences.
- `Implementation guidance:` the intended approach, named patterns, or files to mirror. Label it as guidance unless the approach is genuinely constrained, and say so when it is.
- `Depends on:` blocking story keys or external prerequisites, or `none`.
- `Context:` deep links to the specific design-doc sections, ADRs, or code that define this story. Prefer anchored section links over whole-document links.

When scoping from a design document with workstreams, an optional single trace line such as `Workstream A · Breakdown ref: S1 · Effort: M` may open the description. It counts toward the size limit.

## Engineer Input Needed (Optional)

An optional final labeled line in the description that marks requirement edges where the design does not carry enough concrete code context and the assignee should supply it before or during implementation.

- `Engineer input needed:` name the gap and the exact artifact requested.

Use it when a concrete example would keep the implementing agent from inventing a new solution or exploring the codebase blind. For example, a story adds a repository-populated property to a model; a GitHub permalink to another service that already performs the same lookup anchors the implementation. Limit it to one or two gaps; more than that means the story is under-scoped.

## Acceptance Criteria Contract

- At most three Given/When/Then scenarios.
- Describe observable behavior, business rules, boundary cases, and error outcomes, not implementation steps.
- Include story-specific nonfunctional requirements here, such as `responds within 500 ms`.
- For documentation, research, infrastructure, or purely technical stories, up to three testable declarative bullets are clearer than forced Given/When/Then.
- Do not use `product owner approved` as a substitute for testable criteria.

## Definition of Done Contract

- At most three checklist bullets.
- The first bullet references the team-level Definition of Done standard. Link it when the team has a published one; otherwise use `Standard team Definition of Done satisfied`.
- Remaining bullets are story-specific completion requirements or explicit exceptions only, such as a dashboard update, a runbook change, or a feature-flag cohort.
- Never repeat acceptance criteria or the generic team checklist.

## Acceptance Criteria vs Definition of Done

| Field | Question it answers | Scope |
| --- | --- | --- |
| Acceptance Criteria | Does this behave as the story requires? | Specific to the story |
| Definition of Done | Is this work genuinely complete and safely deliverable? | Shared team standard plus story-specific additions |

Routing guardrails:

- Required system behavior goes in acceptance criteria.
- A story-specific nonfunctional requirement goes in acceptance criteria.
- A recurring team quality or delivery standard belongs to the team-level Definition of Done, referenced rather than repeated.
- Dependencies and references go in the description, never in either checklist.
