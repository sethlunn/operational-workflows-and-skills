# Proposal: Operational Skills Architecture and Governance

- Reviewed artifacts:
  - `README.md`
  - current `codex/*/SKILL.md` adapters
  - current `workflows/*`
  - current `references/*`
  - current `templates/*`
- Proposal date: `2026-04-02`
- Recommendation: `Adopt as the target operating model`

## Short Summary

- The repo is already using the right basic split: thin skill adapters, shared workflows, shared references, and reusable templates.
- The main risk now is not lack of capability. It is uncontrolled growth: too many top-level skills, duplicated workflow text, and blurry boundaries between task entrypoints and reusable procedure.
- The right next step is not "create more skills." It is to standardize skill boundaries, introduce a small taxonomy, and refactor repeated analysis patterns into shared workflow primitives.
- A new skill should exist only when the trigger surface or output contract is materially different. Variants should usually become branch workflows, references, templates, or scripts.
- Near term, the current repo should keep the existing top-level skills but extract a shared `service-analysis` layer used by endpoint and metric analysis workflows.

## Executive Summary

The current repo design is directionally strong. The README already states the right architecture: shared workflows hold the business logic, references hold stable facts, templates hold output shape, and model-specific adapters stay thin.

That is the correct foundation. The problem is that the repo now needs explicit governance so this pattern does not erode as more workflows are added.

The core operating rule should be:

1. a skill is an entrypoint
2. a workflow is the reusable procedure
3. a reference is stable supporting knowledge
4. a template is output shape
5. a script is deterministic execution support

The repo should optimize for:

- a small number of clear entry skills
- reusable shared workflows under those skills
- branch-specific docs only where behavior actually differs
- explicit output contracts and canonical writers
- minimal duplication across skills

This proposal keeps the current top-level skills, introduces sharper taxonomy and maintenance rules, and recommends one immediate refactor family: shared service-analysis primitives used by `service-endpoint-traffic-analysis` and `service-metric-analysis`.

## Current-State Assessment

## What Is Working

- Skill adapters are already thin.
- The repo already distinguishes workflows, references, templates, and scripts.
- `dynatrace-investigation` is already using the correct router pattern: one top-level skill, one shared workflow, several branch playbooks.
- The PagerDuty incident flow already distinguishes orchestrator behavior from bounded child investigation behavior.
- The new `service-metric-analysis` skill follows the same basic pattern and fits the repo shape.

## What Will Become Painful If Left Unchecked

### 1. Too many top-level skills for closely related analysis patterns

The repo now has multiple codebase-plus-telemetry analysis entrypoints:

- `service-endpoint-traffic-analysis`
- `service-metric-analysis`
- parts of `dynatrace-investigation`
- parts of `pagerduty-assigned-service-health`

Those are not duplicates, but they are close enough that shared primitives should exist before more of them are added.

### 2. Workflow duplication risk is higher than skill duplication risk

The real maintenance burden will not come from the small `SKILL.md` files. It will come from:

- repeated "inspect code, identify signals, query Dynatrace, explain caveats, publish to Confluence" text
- repeated environment defaults
- repeated evidence-vs-inference rules
- repeated Confluence-writing rules

If those rules drift, the skills will look separate even when they should behave consistently.

### 3. The repo does not yet have explicit creation rules

Right now, the implied rules are reasonable but not formalized:

- when to create a new skill
- when to branch an existing workflow
- when to extract a reference
- when to create a template
- when to move a repeated operation into a script

That ambiguity creates accidental sprawl.

## Target Operating Model

## Core Principle

Skills should be small, modular, and reusable, but "small" should apply to the adapter layer, not to the number of concepts in the repo.

The goal is not maximum fragmentation. The goal is maximum reuse with clean entrypoints.

The correct hierarchy is:

```text
user request
  -> entry skill
  -> shared workflow
  -> optional branch workflow
  -> references/templates/scripts as needed
  -> final artifact
```

## Layer Definitions

### 1. Skills

Purpose:

- declare when the agent should trigger
- establish the default mode and scope
- route to the correct shared workflow
- point to the minimum required references and templates

A skill should not be the full operating manual.

Rules:

- keep `SKILL.md` small and routing-oriented
- define one clear trigger surface
- link directly to the workflow and only the references/templates that are commonly needed
- avoid embedding long examples, query libraries, or output prose in the skill itself

### 2. Workflows

Purpose:

- define the reusable multi-step procedure
- establish scope discipline
- define the output contract
- define what must be measured, verified, or published

Rules:

- one workflow should describe one procedure family
- if one phase varies materially by scenario, split that phase into branch docs
- include default environment and publishing behavior
- distinguish direct evidence from interpretation
- define the canonical writer when multiple agents or child investigations are involved

### 3. References

Purpose:

- hold stable facts and reusable knowledge, not step-by-step procedure

Examples:

- query patterns
- environment routing
- evidence interpretation rules
- naming conventions
- platform-specific caveats

Rules:

- references should be fact-heavy and procedure-light
- if a reference starts telling the model what sequence of actions to perform, it is probably a workflow

### 4. Templates

Purpose:

- define output shape and expected sections

Rules:

- templates should describe structure, not process
- templates should not duplicate the workflow's decision rules

### 5. Scripts

Purpose:

- support deterministic, brittle, or frequently repeated operations

Rules:

- add scripts only when free-form agent reasoning repeatedly produces the same brittle sequence
- if a script is only saving tokens but not improving reliability, it is probably not worth adding

## Taxonomy Recommendation

The repo should be managed as a small set of skill families rather than as a flat list of unrelated skills.

## Family 1: Incident And Operational Orchestration

Entry skills:

- `pagerduty-incident-analysis`
- `incident-followup-planning`

Characteristics:

- starting object is an incident or incident-derived artifact
- there is usually a canonical parent document
- sub-investigations may run in parallel
- synthesis matters more than one raw query result

## Family 2: Telemetry Investigation Workers

Entry skill:

- `dynatrace-investigation`

Characteristics:

- starting object is a telemetry question
- the skill acts as a router into narrower branch workflows
- can run standalone or as a bounded child investigation

This is the cleanest pattern in the repo and should be treated as the reference model for future branching.

## Family 3: Service Analysis And Documentation

Current entry skills:

- `service-endpoint-traffic-analysis`
- `service-metric-analysis`
- `pagerduty-assigned-service-health`

Characteristics:

- starting object is a service or component
- code inspection and telemetry analysis are both required
- output is usually a Confluence documentation artifact rather than an incident narrative

Recommendation:

- keep the top-level entry skills for now
- extract shared service-analysis workflow primitives underneath them

## Family 4: Review And Triage

Entry skill:

- `babysit-pr`

Characteristics:

- starting object is a PR or review thread
- output is review findings, replies, or bounded fix decisions

This family should stay separate from the operational analysis families.

## Decision Rules For Creating New Artifacts

## Create a New Skill When

- the user starts from a materially different object
- the output contract is materially different
- the mode of operation is materially different
- the trigger sentence would be confusing if folded into an existing skill description

Examples:

- incident id to incident write-up
- service/component to telemetry-backed analysis page
- PR link to review triage

## Do Not Create a New Skill When

- the starting object is the same
- only one branch of the procedure differs
- the same skill could route into a narrower workflow branch cleanly
- the variation is really an output template choice

Examples:

- "service metric analysis for CLC" should not be a new skill
- "Redis incident analysis" should not be a new skill
- "incident analysis but publish to a different Confluence folder" should not be a new skill

## Create a New Workflow When

- the reusable procedure is different enough that one shared workflow becomes unclear

## Create a Branch Workflow When

- the top-level flow is the same
- one phase changes significantly by scenario

This is the `dynatrace-investigation` pattern and should be reused.

## Create a New Reference When

- the information is stable
- multiple workflows need it
- it is facts or patterns, not sequencing

## Create a New Template When

- multiple workflows emit a similar artifact shape
- structure is worth standardizing even if the evidence differs

## Create a Script When

- a step is deterministic, brittle, or repeatedly reimplemented
- the script materially improves reliability or consistency

## Packaging Standard

Every top-level skill should follow the same minimal contract:

1. `SKILL.md`
2. `agents/openai.yaml`
3. one primary workflow reference
4. optional references/templates named explicitly in the skill

Each workflow should declare:

- starting object
- default environment
- default publish behavior
- canonical writer if publishing is involved
- output contract
- evidence vs inference rule
- conditions under which the question may be unanswerable from available telemetry

Each template should be tied to a workflow family, not to one single historical artifact.

## Governance Rules

## 1. Keep Entry Skills Few And Sharp

A new top-level skill should require a written justification:

- what starts the workflow
- why an existing skill cannot route to it
- what output contract is new

## 2. Prefer Shared Workflow Primitives Over New Skills

If two workflows both do:

- code inspection
- telemetry rollup
- caveat analysis
- Confluence publishing

then the shared parts should be extracted before a third similar workflow is added.

## 3. Every Workflow Must Name Its Canonical Writer

This is already explicit in the incident flow and should become standard.

Examples:

- parent incident workflow writes the parent page
- child Dynatrace investigations return evidence packages
- service analysis workflow is the canonical writer for its Confluence page unless a larger orchestrator owns it

## 4. Separate Evidence From Interpretation Everywhere

This rule should not be incident-only.

Every analysis workflow should distinguish:

- direct code evidence
- direct telemetry evidence
- interpretation and inference

## 5. Standardize Measurability Warnings

Service analysis workflows should always distinguish:

- event counts
- distinct-entity counts
- what the metric stream can answer
- what the metric stream cannot answer

This is important enough that it should be a standard rule, not an ad hoc caveat.

## 6. Use Trial And Publish Modes Deliberately

Where possible:

- workflows should default to non-publishing or trial behavior unless the user explicitly asks to publish
- once a workflow is in publish mode, the canonical writer rule should be clear

Not every skill needs two modes, but every publishing workflow should define the default.

## Proposed Near-Term Refactor Plan

## Phase 1: Governance Only

Adopt this proposal as the repo's skill-management rule set without changing the public skill surface yet.

Deliverables:

- this design proposal
- optional later README summary once the team agrees on the conventions

## Phase 2: Extract Shared Service-Analysis Primitives

Create shared workflow and reference material for the common service-analysis pattern.

Recommended extractions:

- `workflows/service-analysis-common.md`
  - resolve target
  - inspect codebase boundaries
  - check local `AGENTS.md`
  - define exact question
  - define environment and publish mode
  - define evidence vs inference standard
- `references/telemetry-measurability.md`
  - event counts vs unique ids
  - dimension coverage gaps
  - scalar-vs-window mismatch handling
  - fixed-window source-of-truth rule
- `references/confluence-analysis-writing-standard.md`
  - absolute timestamps
  - exact filters
  - page caveat placement
  - exact query inclusion rules

Then make:

- `service-endpoint-traffic-analysis`
- `service-metric-analysis`

leaner by pointing at the common workflow plus a branch-specific workflow.

## Phase 3: Normalize Service Analysis As A Family

Reshape the service-analysis family so the pattern is obvious.

Recommended structure:

```text
workflows/
  service-analysis-common.md
  service-endpoint-traffic-analysis.md
  service-metric-analysis.md
references/
  telemetry-measurability.md
  confluence-analysis-writing-standard.md
templates/
  endpoint-traffic-analysis-page.md
  service-metric-analysis-page.md
```

At this phase, decide whether `pagerduty-assigned-service-health` should also consume the shared service-analysis common layer or remain separate.

My bias is:

- keep it separate if it remains ownership-first and code-inspection-light
- converge it later only if it becomes another code-plus-telemetry documentation workflow

## Phase 4: Add Lightweight Quality Gates

Introduce a low-friction review checklist for new skills and workflows.

Every new skill or workflow should answer:

1. what is the starting object
2. what is the output artifact
3. why is a new skill needed instead of a branch
4. what existing references/templates are reused
5. what evidence/inference rules apply
6. what failure or unanswerability modes exist

This can start as a design-review convention before it becomes a script or template.

## Recommendations For Current Skills

## Keep As-Is At The Top Level

- `pagerduty-incident-analysis`
- `dynatrace-investigation`
- `incident-followup-planning`
- `babysit-pr`

Those have clearly distinct starting objects and output contracts.

## Keep, But Refactor Underneath

- `service-endpoint-traffic-analysis`
- `service-metric-analysis`

These should remain separate entry skills because the requested outputs are distinct enough, but they should share more underlying procedure.

## Reassess After More Usage

- `pagerduty-assigned-service-health`

This should stay separate for now, but it is the most likely candidate to either:

- become a more general service-health workflow family
- or consume a shared service-analysis layer if it evolves toward documentation rather than quick health summaries

## Naming Convention Recommendation

Use names that communicate the starting object and the job:

- `pagerduty-incident-analysis`
- `dynatrace-investigation`
- `service-metric-analysis`

Avoid names that encode one specific team, incident, feature, or example unless the skill is truly single-purpose.

## What Success Looks Like

The repo is in a healthy state when:

- new capabilities usually extend an existing workflow family instead of creating a new top-level skill
- `SKILL.md` files stay small and routing-oriented
- common caveats are documented once and reused
- analysis workflows give consistent answers about evidence, inference, and measurability
- Confluence artifacts become more consistent without each workflow re-explaining the same standards

## Recommended Immediate Decision

Adopt this proposal and treat it as the default rule set for future additions.

The first concrete follow-up should be the service-analysis refactor, not because the current state is broken, but because that is the part of the repo most likely to accumulate duplicate workflow text next.
