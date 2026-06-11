# Technical Design Writing Rules

Use this reference when creating or revising a technical design document for a proposed change.

## What A Technical Design Doc Must Answer

A good design doc should let a reader answer these quickly:

- What problem are we solving?
- What outcomes must the change achieve?
- What constraints shape the solution space?
- Which option are we choosing, and why?
- How will we deliver it safely?

If the document cannot answer those questions cleanly, it is not ready.

## Design Docs Are Not The Same As Steady-State Service Docs

Use a technical design doc when the artifact is about a planned change.

Use service documentation when the artifact is about how a system currently works in steady state.

This distinction matters:

- service docs optimize for lookup, understanding, and operations
- design docs optimize for decision-making, tradeoffs, and rollout planning

Do not force one shape into the other.

## Revising Existing Drafts

- Preserve useful stable structure when improving an existing design doc.
- Keep issue links, decision records, rollout notes, and open-question sections unless the user explicitly wants a clean rewrite.
- Replace low-signal filler and stale assumptions, but do not destroy navigation that reviewers are already using.

## Section Rules

### Summary

- Write this last.
- Keep it short enough that a reader can decide in under 30 seconds whether they found the right document.
- Connect the problem, chosen solution, and intended outcome.

### Goals

- Translate business intent into technical goals.
- State what the design is trying to deliver, not how it will be implemented.

### Problem Statement

- Describe the current pain, gap, or risk.
- Keep it solution-free.
- Include both customer-facing and internal operational pain when both matter.

### Target Outcomes

- Prefer measurable outcomes.
- Include technical outcomes when they are the only practical expression of success.
- Make it obvious what would count as success or failure.

### Requirements And Constraints

Capture both the stated requirements and the ones that become obvious only after technical analysis.

Functional requirements often include:

- user-visible behaviors
- data shape changes
- integration requirements
- downstream compatibility

Non-functional requirements often include:

- availability
- latency or throughput
- security and privacy
- compliance and policy constraints
- observability and on-call readiness

Also check for delivery and day-two requirements:

- reversibility
- migration, backfill, and reconciliation
- feature-flag management and cleanup
- support or CX visibility
- financial correctness

### Out Of Scope

- Name the things this change will not solve.
- Use this to protect the document from scope creep and to prevent reviewers from inventing missing requirements.

### Stakeholders

- List implementing teams, consuming teams, consult partners, and informed groups.
- Use this to expose coordination risk early.

### Current State

- Reconstruct the current behavior from code, docs, and production evidence.
- Use diagrams when they clarify the existing shape.
- Prefer exact anchors over tribal-memory claims.

### Diagrams

- Use diagrams only when they materially improve understanding.
- Prefer one focused flow or sequence diagram over a broad architecture picture with every adjacent system.
- Keep diagrams aligned with the actual decision boundary. If the design only changes one path, diagram that path.

### Solution Options

- Show meaningful alternatives, not just the chosen answer.
- Compare them against objective criteria derived from the goals and requirements.
- Keep the comparison concise but real.

### Recommended Solution

- Describe the future state, boundaries, important interfaces, and operational expectations.
- Call out unknowns and spike areas directly.
- Include observability, failure handling, and ownership implications when they matter.

### Transition Plan

- Explain how the design gets from current state to future state.
- Cover rollout sequencing, migration, reversibility, coordination points, validation, and cleanup.
- If the transition plan is weak, the design is incomplete even when the future state sounds correct.

## Review Mode

When the task is to review a design doc rather than draft one:

- lead with findings, not with a rewritten summary
- prioritize gaps that would change implementation risk, coordination, or rollout safety
- separate missing evidence from disagreements about style
- propose replacement wording only after the findings are clear

## Mixed-Mode Rule

Technical design docs are a deliberate mixed-mode exception to the repo's usual Diataxis split.

That is acceptable because the reader needs:

- explanation to understand the problem and tradeoffs
- scoped reference to understand the concrete contracts and constraints
- transition planning to understand delivery and rollout risk

Keep those modes intentional and sectioned. Do not let the document become an unstructured scratch pad.

## Review Standard

Before calling a design doc solid, check:

- Is the problem statement distinct from the chosen solution?
- Are the target outcomes explicit?
- Are functional and non-functional requirements both covered?
- Are rollout, reversibility, migration, and observability addressed where relevant?
- Are alternatives compared against objective criteria?
- Is there an explicit transition plan?
- Are open questions named instead of hidden?
