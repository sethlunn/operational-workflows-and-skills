# Architecture Decision Record Writing Rules

Use these rules when creating, reviewing, revising, or repairing an Architecture Decision Record (ADR). They are based on a read-only review of all 14 records in the [QD ADR folder](https://quadpay.atlassian.net/wiki/spaces/QD/folder/5711560731/ADRs), with deeper comparison of compact and drifted examples.

## What An ADR Is

An ADR is a durable record of one consequential architectural choice: the problem, the criteria, the alternatives, the selected option, and the expected effects. It is not the complete design, implementation plan, runbook, onboarding guide, project tracker, or research notebook.

Use a declarative title such as `Use X for Y` or `X as Y`. Keep one independently reversible decision per ADR. Split the record when it contains multiple choices that could change on different timelines or for different reasons.

## QD House Format

All 14 reviewed ADRs use the same MADR-style core in this order:

1. Context and Problem Statement
2. Decision Drivers
3. Considered Options
4. Decision Outcome
   - Consequences
   - Confirmation
5. Pros and Cons of the Options
   - One subsection per option

All 14 use typed comparison bullets such as `Good, because ...`, `Bad, because ...`, and occasionally `Neutral, because ...`. Preserve this recognizable structure rather than inventing a new one for each record.

Add compact lifecycle metadata above the core sections:

- ADR ID
- Status: `Draft`, `Proposed`, `Accepted`, `Rejected`, `Superseded`, or `Deprecated`
- Date
- Decision owner
- Reviewers, when known
- Jira or initiative link
- `Supersedes` and `Superseded by`, when applicable

Confluence page status such as `current` describes the page, not the architectural decision. The body metadata is authoritative and must agree with the outcome wording.

## Canonical Section Rules

### Context and Problem Statement

Target 75–150 words. State the current situation, pressure, scope boundary, and decision question in neutral terms. Do not preload the section with the preferred implementation or reproduce the design history.

### Decision Drivers

Use 3–7 concise criteria that materially distinguish the options. Drivers should express outcomes or constraints, not features of the favored option. Order them by importance when priorities are known.

### Considered Options

List 2–4 peer alternatives as one-line names. Include the status quo or no-central-solution option when it is viable. Use the exact same names in Decision Outcome and Pros and Cons.

### Decision Outcome

Target 50–175 words and one or two paragraphs. Begin with one unambiguous sentence:

- Accepted: `Chosen option: "<exact option name>", because <top drivers>.`
- Draft or Proposed: `Proposed option: "<exact option name>", because <top drivers>.`

Explain only why the option wins and any essential scope boundary, exception, or fallback. Do not add nested design sections, command examples, permission matrices, interface schemas, delivery sequences, or a dated diary of refinements.

### Consequences

Use 3–7 `Good`, `Bad`, or `Neutral` bullets about the selected option only. Include adoption cost, ownership, operational burden, lock-in, or residual risk when material. Do not duplicate the full comparison of every option.

### Confirmation

Use 1–5 observable checks or decision gates that can confirm or disprove the choice. Name an owner or linked evidence when available. Confirmation is not a rollout plan, compliance backlog, or POC task list.

### Pros and Cons of the Options

Create one heading per exact option name. Give each peer option symmetrical treatment with 1–3 meaningful advantages and 1–3 meaningful disadvantages, using the QD typed-bullet convention. Keep the analysis neutral and evidence-oriented; avoid promotional claims.

### References

This optional final section contains links and one-line annotations only. Put changing evidence in living companion documents and link it here.

## Size And Extraction Guardrails

The reviewed folder has a median of about 676 plain-text words. Its clearest records are roughly 450–800 words; bloat is visible in records above 1,000 words, especially when Decision Outcome becomes a design specification.

- Target 450–900 words.
- Trigger a drift review above 1,000 words.
- Always trigger a split review when the ADR is roughly twice the folder median, has many non-option subsections, or makes more than one independently reversible choice.
- Prefer extraction over compression when detail remains valuable.

Keep in the ADR:

- one decision question and scope boundary
- stable differentiating drivers
- peer alternatives and the selected option
- essential boundaries and exceptions
- material positive, negative, and neutral consequences
- concise confirmation conditions
- lifecycle and supersession metadata

Move to linked companion documents:

- architecture, interfaces, schemas, detailed flows, permission matrices, and package contracts → technical design or specification
- operational steps, commands, diagnostics, and failure recovery → runbook
- persona guidance and exact client walkthroughs → onboarding guide
- vendor capability and client-support matrices → living compatibility reference
- research snapshots and dated evidence → evidence page
- experiments, acceptance scripts, and verification spikes → POC or test plan
- migration tables and delivery sequencing → rollout guide
- open questions, work items, asks, and roadmap → Jira or project plan
- contribution and review tiers → contribution or security policy

Remove rather than extract:

- duplicate explanations
- stale or superseded walkthroughs
- historical refinement prose already represented by page history
- unverified statements presented as fact
- detail that neither explains the choice nor changes a consequence

## Lifecycle And Revision Policy

Use this lifecycle:

`Draft → Proposed → Accepted | Rejected → Superseded | Deprecated`

### Draft Or Proposed

- Rewrite canonical sections in place; do not append a second interpretation.
- Replace stale details and resolve contradictions.
- Keep the decision question stable. Split the ADR if its scope becomes multiple decisions.
- Keep the status `Proposed` while a decision-blocking assumption is unresolved.
- Record every substantive Confluence edit with a meaningful version message.
- Ask the decision owner and affected stakeholders to review it before acceptance.

### Accepted

Allow only spelling and formatting corrections, link repair, clearer wording that does not change meaning, and updated confirmation evidence. Do not silently change the decision question, drivers, scope, selected option, or material consequences.

When new evidence changes the decision, create a successor ADR. Mark the old record `Superseded`, add `Superseded by`, and add the reciprocal `Supersedes` link to the new record. Implementation evolution that does not change the architectural choice belongs in the companion design or runbook.

### Rejected, Superseded, Or Deprecated

Preserve the historical body. Update only lifecycle metadata, confirmation evidence when relevant, and links. Do not rewrite history to make the old decision appear current.

## Drift-Repair Workflow

Before editing, capture the decision invariant:

- exact decision question
- lifecycle status
- exact selected or proposed option
- top drivers
- scope boundary
- decision owner

If those cannot be recovered confidently, produce an audit and questions rather than silently inventing a cleaner decision.

Then:

1. Read the current body and useful version history before drafting changes.
2. Inventory headings, approximate word count, repeated claims, contradictions, time-sensitive facts, and embedded non-ADR artifacts.
3. Classify each useful statement into one canonical ADR section or a named companion-document destination.
4. Produce a `retain / move / remove` plan before a material rewrite.
5. Rewrite Draft or Proposed sections in place. Do not preserve obsolete prose merely because it existed.
6. For an Accepted ADR, stop and propose a successor if the invariant would change.
7. Run the consistency audit below.
8. When publishing, use a meaningful Confluence version message and reread the rendered page for malformed headings, tables, links, or macros.
9. Report what was retained, moved, removed, and whether the word and section budgets improved.

Never update Confluence unless the user explicitly asks. For a review-only request, return findings and proposed wording without publishing.

## Consistency And Drift Audit

- Is there exactly one architectural decision?
- Are the canonical sections present and ordered correctly?
- Can the outcome be understood in one or two paragraphs?
- Is the chosen option one of the considered options?
- Are option names identical in every section?
- Does every driver materially distinguish the options?
- Is the status quo represented when meaningful?
- Do Consequences discuss the selected option rather than repeat all option analysis?
- Is Confirmation observable rather than a task backlog?
- Are assumptions visibly distinct from confirmed facts?
- Are decision-blocking unknowns consistent with `Draft` or `Proposed` status?
- Are time-sensitive vendor capabilities in living references rather than the permanent decision core?
- Do any sections contradict one another?
- Are implementation steps, walkthroughs, roadmaps, or project asks embedded?
- Is material repeated across sections?
- Do size or heading count indicate a split review?
- Do lifecycle metadata and outcome wording agree?
- Does the requested edit require a successor rather than an in-place change?
- Are supersession links reciprocal?
- Does the Confluence edit have a meaningful version message?
- Was the rendered page checked after publishing?

## Current Tooling ADR Drift Baseline

The [Session Tooling ADR](https://quadpay.atlassian.net/wiki/spaces/~5f57c79ed2c77e0075a51ea7/pages/5737938952/) at version 6 is approximately 44,292 Confluence storage characters, about 6.5 times the folder median, and has 29 headings. It retains the core format but then expands into contributor governance, package and MCP specifications, review policy, onboarding walkthroughs, vendor matrices, migration guidance, a roadmap, a POC plan, open questions, and platform asks.

It also contains additive-edit contradictions. For example, the decision removes feed authentication from the default path and redefines `onboard_me` as diagnostics, while a retained walkthrough still has `onboard_me` install local files and authenticate to Azure Artifacts. It describes a workspace-provided Codex plugin as installed or one-click while leaving that behavior as an open verification question.

A future repair should preserve the core package/MCP boundary and its drivers, then extract the specifications, onboarding, compatibility evidence, contribution policy, rollout, and project tracking into linked living documents. It should delete duplicated and superseded prose rather than appending another dated refinement. Because the ADR is still proposed, its canonical sections may be coherently rewritten in place after the user approves a retain/move/remove plan.
