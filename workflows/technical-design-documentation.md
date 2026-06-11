# Technical Design Documentation

Draft or revise a technical design document for a planned change. Use this when the artifact must explain why a change exists, what must be true for it to succeed, which solution wins, and how it will be delivered safely.

Read `../references/technical-design-writing-rules.md` before drafting.
Read `../references/diataxis-writing-rules.md` when you need the repo's broader document-type rules.
Read `../templates/technical-design-document.md` when writing the final document.
Read `../references/confluence-routing.md` when publishing to or updating Confluence.

## Defaults

- Default to `trial mode`.
- Do not create or update Confluence pages unless the user explicitly asks.
- Treat the design doc as the canonical artifact for the proposed change.
- If the user actually needs steady-state system docs, route to `../workflows/service-system-documentation.md` instead of forcing design material into that doc set.
- When revising an existing draft, preserve useful stable sections, issue links, and decision records unless the user explicitly asks for a clean rewrite.

## Strong Inputs

Prefer gathering as many of these as are available:

- Jira story, epic, or initiative
- PRD, proposal, or architecture page
- existing Confluence design draft when revising
- existing Confluence page URL or page id when revising or publishing
- repo paths and code anchors for the current state
- rollout constraints, dependencies, and environment assumptions
- telemetry or incident evidence when the design is responding to production behavior

## Workflow

1. Resolve the document job and output mode.
- Confirm whether the task is:
  - draft a new design doc
  - improve an existing draft
  - review a design doc for gaps
  - convert scattered notes into a design doc
- Decide whether the output should stay in session, become local markdown, or update Confluence.
- Record the exact change scope so the doc does not sprawl into adjacent projects.
- If the task is a review, decide whether the user wants:
  - findings only
  - findings plus proposed revised wording
  - a fully rewritten document

2. Build the requirement context before writing prose.
- Read the Jira story, parent, epic, PRD, and any linked Confluence proposal pages.
- Reduce them into the design pressures that matter:
  - problem statement
  - goals
  - target outcomes
  - explicit non-goals
  - functional requirements
  - non-functional requirements
  - rollout, compliance, or coordination constraints
- If the requirement sources disagree, name the conflict instead of silently choosing one.

3. Reconstruct the current state from evidence.
- Inspect code, configuration, existing docs, and operational evidence that define the current system shape.
- Capture:
  - current workflow or system behavior
  - current integrations and data movement
  - current failure modes or operational pain
  - current ownership or team boundaries
- Prefer direct evidence and exact anchors over reconstructed lore.

4. Expand the hidden requirements.
- Do not stop at the obvious product requirements.
- Check whether the design must also account for:
  - observability and alerting
  - reversibility and rollback
  - migration, backfill, or reconciliation
  - feature flags and cleanup work
  - support, CX, or Nexus visibility
  - data retention, clearing, or financial correctness
  - security, privacy, or policy constraints
  - day-two operability after rollout

5. Compare solution options against objective criteria.
- Enumerate the meaningful options, including keeping the current state when useful.
- Choose comparison criteria from the actual problem instead of using generic prose. Typical criteria include:
  - delivery risk
  - reversibility
  - operational visibility
  - migration complexity
  - latency or throughput impact
  - ownership clarity
- For each option, capture:
  - summary
  - benefits
  - costs and risks
  - constraints or unknowns
  - why it does or does not satisfy the requirements
- Do not jump straight to the preferred implementation without showing why it wins.

6. Write the recommended design clearly.
- Describe the future-state architecture, flow, and important contracts.
- Call out:
  - boundaries and ownership
  - user-visible behavior changes
  - interfaces, schemas, or event changes
  - failure handling
  - telemetry and alerting expectations
  - open questions and spike areas
- Use one or two focused diagrams only when they materially improve comprehension. Prefer a focused flow or sequence over a full-system architecture map.

7. Build the transition plan explicitly.
- Include the delivery shape, not just the steady-state destination.
- Cover:
  - sequencing
  - coordination points
  - rollout and reversibility
  - migration or backfill steps
  - test and validation expectations
  - definition-of-done cleanup work
- If another team must act before the change is safe, name that dependency directly.

8. Write the summary last.
- Keep it to one or two sentences that connect:
  - the problem
  - the chosen solution
  - the expected outcome
- A reader should know within 30 seconds whether this is the right document.

9. Finalize with gaps and evidence boundaries.
- Separate direct evidence from interpretation.
- State exact dates, environments, and systems inspected when relevant.
- Name unresolved questions instead of smoothing them over.
- Trim raw scratch notes out of the final doc unless they materially support the design decision.

## Context And Command Patterns

Prefer these Atlassian reads:

- `searchAtlassian` to locate the Jira story, PRD, or existing Confluence draft when the user does not give an exact key or page id
- `getJiraIssue` for requirements, acceptance criteria, and rollout notes
- `getConfluencePage` for the current draft or related proposal page

Prefer these local checks:

- `rg` for implementation anchors, interface definitions, and config surfaces that prove the current state
- focused reads of controllers, handlers, schemas, migrations, feature-flag wiring, pipeline files, and deployment manifests when they materially shape the design

Prefer these operational reads when needed:

- Dynatrace evidence only when current production behavior materially affects the design decision
- incident or postmortem evidence when the design is a response to a known failure mode

Prefer these publishing rules:

- when revising an existing Confluence page, update by known page id or URL rather than title search
- when creating a local markdown draft, prefer `reviews/design/<design-slug>/`

## Output Rules

- Use `templates/technical-design-document.md` for the final artifact.
- Technical design docs are an intentional mixed-mode artifact: explanation, scoped reference, and transition planning may all appear in one document when they serve the design decision.
- Keep the problem statement free of implementation bias.
- Keep target outcomes measurable where possible.
- Keep requirements solution-agnostic unless the product or compliance requirement is already implementation-specific.
- Keep the options section comparative and decision-oriented, not a dumping ground of brainstorming.
- Keep the transition plan concrete enough to expose rollout risk, coordination cost, and cleanup work.
- When revising an existing design doc, preserve useful stable sections such as issue links, approvals, open questions, and rollout notes unless a rewrite is explicitly requested.
- When the task is a review, present findings first and draft replacement prose second.
