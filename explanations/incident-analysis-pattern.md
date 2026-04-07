# Incident Analysis Pattern

This page explains why the PagerDuty incident workflow is shaped the way it is.

Use `workflows/pagerduty-incident-analysis.md` when you need to run the procedure. Use this page when you need the rationale behind that procedure.

## Why Trial Mode Is The Default

The workflow defaults to `trial mode` because investigation and publication are different actions.

Most incident requests begin as a request to inspect, not a request to publish. Keeping `trial mode` as the default lets the workflow gather evidence, build a parent-page-ready result, and avoid external writes until the user explicitly asks for them.

That keeps the normal path safer and reduces the chance of publishing a page before the investigation is coherent.

## Why The Parent Surface Exists Early

The workflow creates the parent investigation surface before deep child investigation work.

This matters because the parent surface is the place where the exact window, ownership, scope, investigation queue, and eventual synthesis live. Without that shared parent, child tracks tend to drift into separate narratives, inconsistent scopes, or duplicated write-ups.

In `publish mode`, that parent surface is the Confluence page. In `trial mode`, it is an in-memory parent-page-ready result.

## Why The Workflow Starts With A High-Level Sweep

The high-level Dynatrace sweep is not supposed to prove every root-cause claim on its own.

Its job is to answer a narrower set of questions first:

- what looks broken
- when it first looks clearly broken
- whether the blast radius appears broad or narrow
- which deeper tracks are worth the cost

That first pass exists to shape the investigation queue, not to replace later bounded evidence gathering.

## Why Child Investigations Stay Narrow

Child investigations are intentionally bounded.

Each child track should answer one exact question in one time window for one scoped path, dependency, or identifier chain. That makes the returned evidence easier to audit and makes contradictions easier to notice.

When child tracks try to explain the entire incident, they usually blur scope boundaries and duplicate the parent workflow.

## Why The Parent Workflow Is The Canonical Writer

The parent workflow owns the final incident narrative.

Child tracks can contribute evidence packages, but they should not each try to author the main incident page independently. A single canonical writer is what allows the final page to reconcile contradictory evidence, state the strongest explanation, and keep one authoritative summary.

This is why the workflow prefers child-result contracts over ad hoc notes.

## Why Retrospective Cleanup Matters

Live incident language and final incident language are not the same.

During an active incident, phrases such as `ACTIVE`, `still resolving`, or `best read so far` can be useful. Once the incident is resolved, those phrases become stale and can mislead later readers.

Retrospective cleanup exists to:

- refresh final timestamps and status
- separate onset evidence from late secondary activity
- remove stale live-state wording
- restate the conclusion in final tense

Without that pass, even a technically correct page can still feel operationally unreliable.
