# Service Documentation Pattern

This page explains how service or system documentation should be produced in this repo using Diataxis.

Use `workflows/service-system-documentation.md` when you need to run the process. Use this page when you need the rationale behind the doc set, the terminology, or the recommended pilot shape.

## The Core Rule

Do not aim for one giant "system documentation" page by default.

For most services, the better shape is a small documentation set:

- one explanation overview
- one reference page
- one operability guide
- one tutorial only when there is a real guided first-run need

That keeps each artifact aligned to one reader need instead of forcing architecture, inventories, and runbooks into one mixed page.

## What Reference Means Here

Reference is curated lookup material, not all discovered context.

Good reference examples:

- endpoint inventory
- event and queue inventory
- schema catalog
- runtime entity map
- dashboard, alert, and SLO anchors

Bad reference examples:

- raw code crawl notes
- untrimmed telemetry observations
- Confluence comment threads
- page version history

Those are evidence or scratch surfaces, not final reference docs.

## What Template Means Here

A template is the output contract for a final artifact.

Templates define the structure of the final docs the parent workflow writes. They do not define where every child agent should dump raw findings, and they are not the same thing as reference.

In this pattern:

- `templates/service-overview-page.md` is the explanation output contract
- `templates/service-reference-page.md` is the reference output contract
- `templates/service-operability-guide.md` is the how-to output contract

When a trial-mode run needs local files, prefer placing the generated doc set under `reviews/service-docs/<service-slug>/` so the pages stay grouped without being mistaken for stable reference.

When a publish-mode run creates or updates Confluence pages, store the stable page ids alongside the draft set in `reviews/service-docs/<service-slug>/confluence-manifest.yaml`.

## What Child Agents Should Produce

Child agents should return bounded evidence packages.

Those packages can live in:

- local markdown scratch
- temporary Confluence notes
- reusable child-result files

But they should be treated as:

- evidence
- scratch
- intermediate context

They become reader-facing documentation only after the parent synthesizes and trims them into the final doc set.

## What Goes Where

- Mermaid diagrams:
  - explanation when they show conceptual architecture or flows
  - reference when they show exact topology or catalog-style mappings
- Data schemas:
  - reference
- Call stacks:
  - reference when exact and stable
  - operability guide when they support a debugging procedure
- Traffic analysis:
  - reference when it is inventory-like
  - explanation when it clarifies why the system behaves a certain way
- Debug steps:
  - operability guide

## Why Subagents Fit Well

Service documentation is a strong subagent candidate because the discovery tracks often split naturally.

Good bounded tracks:

- code structure and runtime entrypoints
- interfaces and contracts
- topology and telemetry
- operability and debugging guidance

The parent should remain the canonical writer for the final doc set.

## Current Worked Example

The first concrete trial-mode doc set now lives under:

- `reviews/service-docs/provider-gateways-machine-learning-gateway/`

That folder now also includes a `confluence-manifest.yaml` file that records the published Confluence page ids for deterministic updates.

It is a useful worked example because:

- it has a bounded single deployable
- it mixes legacy Databricks routes with a newer fan-out path
- it has service-definition SLO metadata and a broader Confluence anchor
- it forces the doc set to separate architecture, exact route inventory, and operational debugging guidance

That makes it a better terminology test than a smaller toy service and a safer first pass than a much larger multi-surface system.
