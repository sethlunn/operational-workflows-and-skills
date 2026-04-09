# Service System Documentation

Produce a Diataxis-aligned documentation set for a repo service or larger service surface by combining code-backed structure discovery with operational telemetry evidence.

Read `../workflows/service-analysis-common.md` before starting.
Read `../references/subagent-usage.md` when deciding whether to split the work into bounded child tracks.
Read `../templates/analysis-child-result.md` when bounded child investigations are being used.
Read `../templates/service-overview-page.md` when writing the explanation-style service overview.
Read `../templates/service-reference-page.md` when writing the reference-style service or system reference.
Read `../templates/service-operability-guide.md` when writing the how-to-style operability guide.
Read `../explanations/service-documentation-pattern.md` when you need the rationale behind the documentation set, the terminology split between evidence and reference, or the recommended pilot shape.

## Subagent Posture

- Strong fit, but bounded.
- Good split patterns:
  - code structure and runtime entrypoints
  - interface and schema inventory
  - Dynatrace topology, traffic, and service health
  - operability and debugging guidance
- Keep the parent as the canonical writer for the final documentation set.
- Avoid splitting small services or highly localized questions.
- When split, have each child return `../templates/analysis-child-result.md`.
- Child outputs may live in local markdown scratch files or temporary Confluence notes when needed, but treat those as evidence packages or scratch surfaces, not as final reference docs.

## Workflow

1. Resolve the target, audience, and operating mode.
- Start from the service or system name the user gives, usually under `services/<service-name>`.
- Write down the exact documentation job:
  - service overview
  - service reference
  - operability guide
  - full documentation set
  - larger system documentation spanning multiple services
- Decide whether the request is for `trial mode` or `publish mode`.
- Default to `trial mode` unless the user explicitly asks to create or update Confluence pages.
- Decide the initial doc set:
  - explanation overview
  - reference page
  - operability guide
  - tutorial only when there is a safe repeatable onboarding path worth teaching

2. Resolve the current documentation surface.
- Inspect `service.json` when present.
- Record:
  - display name
  - existing Confluence URL
  - named microservices or deployables
  - any SLO or ownership metadata
- Inspect any existing local `README`, `docs/`, or service-level documentation before drafting replacements.
- If the user provides an explicit Confluence target, treat it as canonical unless there is a strong reason not to.

3. Build the code-backed understanding first.
- Inspect the files that establish runtime shape:
  - `Program.cs`
  - `Startup.cs`
  - controllers
  - bus handlers
  - domain events
  - data contexts and repositories
  - Helm values
  - `service.json`
  - pipeline files when deployment shape matters
- Record:
  - deployables
  - ingress types such as HTTP, worker, middleware, or projections
  - important outbound dependencies
  - data stores
  - configuration or feature-flag surfaces
  - files that best prove the core execution paths

4. Decide whether to split into bounded child tracks.
- Keep the work in one thread when the service is small enough to stay coherent.
- Split only when there are independent tracks that can be investigated separately.
- Typical child tracks:
  - runtime structure and entrypoints
  - interface and contract inventory
  - topology, traffic, and telemetry evidence
  - operability and debugging guidance
  - data stores and schemas
- Give each child:
  - one exact question
  - one exact scope
  - one reason the track exists

5. Run child investigations and collect evidence.
- Use local code inspection for code-backed tracks.
- Use Dynatrace for:
  - entity discovery
  - traffic or volume
  - dependency topology
  - active or recent problem surfaces
  - key telemetry that supports operational guidance
- Use `../workflows/dynatrace-investigation.md` when a child track becomes a true Dynatrace investigation rather than a lightweight lookup.
- Have each child return:
  - exact question answered
  - exact scope searched
  - strongest direct evidence
  - interpretation
  - unresolved gap
  - parent-ready summary

6. Build the documentation set, not one dumping-ground page.
- Explanation overview:
  - what the service or system does
  - boundaries
  - runtime shape
  - major flows
  - dependency model
  - conceptual diagrams
- Reference page:
  - endpoints
  - events and message contracts
  - schemas and data stores
  - runtime entities
  - ownership, dashboards, alerts, or config surfaces
  - exact file or query anchors
- Operability guide:
  - how to validate health
  - how to assess a rollout
  - how to trace a request, event, or identifier
  - common failure modes
  - exact signals and queries
- Tutorial:
  - only create when there is a safe, repeatable first-run path that is materially useful to readers

7. Place diagrams and artifacts where they serve the reader need.
- Put conceptual flow diagrams in the explanation overview.
- Put endpoint, event, schema, and entity inventories in the reference page.
- Put rollout checks, debugging flows, and trace steps in the operability guide.
- Use traffic analysis as:
  - reference when it is inventory-like
  - explanation when it is clarifying why the system behaves a certain way
- Do not force every diagram, schema, and query into one page.

8. Write or update the output surfaces.
- In `trial mode`:
  - return page-ready markdown for each requested document
  - or create local draft files when the user asks for file output
  - when creating local draft files, prefer `reviews/service-docs/<service-slug>/`
- In `publish mode`:
  - update the existing Confluence page when one is clearly canonical
  - otherwise create a small documentation set with one landing page plus linked child pages when that is the clearer structure
  - record the resulting Confluence folder or page ids in `reviews/service-docs/<service-slug>/confluence-manifest.yaml` so later updates can target stable ids instead of title search
- Keep one canonical writer for the final doc set.

9. Finalize with explicit caveats.
- Include exact dates, scopes, and environments.
- Separate:
  - direct evidence from code
  - direct evidence from telemetry
  - interpretation
- Say explicitly when a question could not be answered from the available code or telemetry.
- Trim scratch evidence out of the final docs unless it materially supports the reader-facing artifact.

## Output Rules

- Prefer a small Diataxis-aligned documentation set over one sprawling catch-all page.
- Do not treat raw child outputs as final reference docs.
- Use exact file paths, exact dates, exact entity ids, and exact query shapes when they materially support the result.
- Keep the explanation overview conceptual, the reference page exact, and the operability guide procedural.
- If the service already has partial documentation, preserve the useful stable facts and replace low-signal narrative rather than rewriting everything blindly.
- Link the created or updated Confluence pages in the final response when publishing is part of the task.
