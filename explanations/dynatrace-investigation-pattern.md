# Dynatrace Investigation Pattern

This page explains why the Dynatrace investigation workflow is shaped as a top-level router with narrow branch playbooks and bounded child investigation rules.

Use `workflows/dynatrace-investigation.md` when you need to run the procedure. Use this page when you need the rationale behind that procedure.

## Why The Workflow Starts With A Router

Dynatrace questions often sound similar at the start and then diverge quickly.

A rollout check, an incident investigation, a service-debugging session, and an identifier trace can all begin with the same sentence shape: "help me understand what happened." The evidence that matters first is not the same in each case.

The router exists to force an early branch choice before the investigation widens into an unfocused search.

## Why The Workflow Chooses One Branch

The workflow tells the reader to choose exactly one branch playbook after the shared preflight.

That constraint exists to prevent one investigation from silently becoming four overlapping investigations. Once the question is narrow enough, one branch should dominate. If another branch becomes necessary later, that should happen because new evidence changed the question, not because the workflow started broad by default.

## Why Narrow Entity Scope Comes Early

Entity resolution happens before expensive drill-downs because the wrong scope can make good queries look empty and bad queries look plausible.

Resolving the narrowest useful entity early reduces broad scanning, separates prod from non-prod, and makes later conclusions easier to defend.

## Why Runtime Shape Matters

The workflow classifies the runtime shape before choosing evidence.

That is because the first useful signal for an HTTP service is often different from the first useful signal for a queue consumer, scheduled job, or telemetry-validation path. Runtime shape is what keeps the workflow from applying the same evidence order to every problem.

## Why The First Evidence Source Is Chosen Deliberately

The workflow asks for one first evidence source that matches the question.

This reduces wasted drill-downs and helps the investigator answer a coarse question before diving into detail. The goal is usually to decide whether a regression is real, where it seems to sit, and which hypothesis deserves the next query.

## Why Queries Stay Narrow

The workflow prefers one concrete hypothesis per query.

Large mixed-purpose queries can return noisy results that are harder to interpret and harder to reuse in a final write-up. Narrow queries make it easier to tell whether a missing result is real absence, wrong scope, wrong field choice, or a telemetry gap.

## Why Child Investigations Must Stay Bounded

When this workflow is used under a parent orchestrator, it should answer one narrow question and return one evidence package.

That keeps the child investigation useful to the parent without turning the child into a second orchestrator. The child can be precise about scope, evidence, and unresolved gaps because it is not trying to explain the entire incident.

## Why Output Rules Separate Evidence And Interpretation

Telemetry often supports a conclusion indirectly rather than directly.

Separating direct evidence from interpretation keeps the workflow honest about ambiguity. It also makes later synthesis easier, because another workflow can reuse the evidence without inheriting stronger claims than the signal really supports.
