# Explanation

- Topic: How the repo's incident-analysis family fits together
- Core question answered: Why incident analysis is split across a PagerDuty orchestrator, a reusable Dynatrace worker, parent and child output contracts, and a separate follow-up planner
- Audience: Contributors extending or maintaining the incident-analysis workflows and skills

# Context

- Why this exists:
  - Incident work in this repo has two jobs at once:
    - determine what happened with defensible evidence
    - leave behind an auditable artifact that can drive follow-up work
- What problem it addresses:
  - Without an explicit family model, incident work tends to collapse into one giant workflow that mixes PagerDuty routing, telemetry exploration, live note-taking, final write-up, and Jira follow-up creation in one place.
- What would be confusing without this explanation:
  - `pagerduty-incident-analysis` and `dynatrace-investigation` both talk about incidents.
  - The repo has both incident templates and follow-up story templates.
  - `trial mode` and `publish mode` can look like two different investigations when they are really two output postures for the same investigation.

# Mental Model

- Main concepts:
  - PagerDuty provides the incident envelope:
    - incident id
    - primary service ownership
    - urgency and status
    - created and resolved timestamps
    - duplicate or related incident context
  - The parent incident workflow turns that envelope into:
    - one exact investigation window
    - one high-level Dynatrace sweep
    - one bounded child-investigation queue
    - one canonical incident narrative
  - Dynatrace investigation is the reusable evidence worker:
    - it narrows scope
    - chooses one branch playbook
    - returns evidence packages
  - Templates are contracts:
    - the parent template defines the final incident artifact shape
    - the child template defines what a bounded track must return
  - Follow-up planning is downstream of incident analysis:
    - it validates the finished incident conclusions
    - it turns validated conclusions into actionable Jira work
- Important boundaries:
  - PagerDuty incident orchestration versus Dynatrace evidence gathering
  - parent page ownership versus child evidence packages
  - investigation posture versus publication posture
  - incident analysis versus follow-up planning
  - thin skill wrapper versus shared workflow versus template contract
- What readers should keep distinct:
  - `trial mode` is full investigation without external writes
  - `publish mode` is the same investigation with an early live Confluence page
  - the high-level sweep narrows the queue; it does not replace deeper evidence gathering
  - a child investigation answers one exact question; it does not own the whole incident story

# Tradeoffs

- Benefits:
  - The parent workflow keeps one canonical narrative instead of several conflicting write-ups.
  - The Dynatrace worker is reusable across incidents, service debugging, rollout checks, and identifier tracing.
  - Defaulting to `trial mode` lowers the risk of publishing a weak incident page too early.
  - Structured child-result contracts make contradictions and missing evidence easier to audit.
  - A separate follow-up planner prevents live triage wording from turning directly into Jira commitments without validation.
- Costs:
  - The family is spread across more files than a monolithic workflow.
  - Contributors need to understand the router, branch playbooks, and templates, not just the entry skill.
  - Strong parent-child discipline is required or the family degrades into overlapping narratives.
- Alternatives considered:
  - one giant incident workflow that directly handles every telemetry branch and all publication steps
  - child investigations writing directly to the main Confluence page
  - creating Jira follow-up work directly from the live incident workflow
  - publishing only at the very end instead of creating the parent surface early in publish mode
- Why this shape was chosen:
  - The family optimizes for bounded evidence, one canonical writer, and safe default behavior.
  - Early parent-surface creation preserves scope, ownership, and queue state while child work is still in progress.
  - Separating follow-up planning from the live investigation makes it easier to validate claims after the incident dust settles.

# Implications

- Operational consequences:
  - Incident runs should converge on a parent-page-ready result even when nothing is published.
  - Adding a new Dynatrace child track means updating the evidence path and preserving the child-result contract.
  - Resolved incidents should receive retrospective cleanup before the write-up is treated as final.
  - Duplicates, composite alerts, and routing noise should be treated as normal cases, not as workflow exceptions.
- Documentation consequences:
  - Workflow files own the operational procedure.
  - Templates own output structure.
  - References own stable lookup rules, routing facts, and lessons learned.
  - Explanation files own the rationale for why the family is split this way.
  - Extending the family usually means touching more than one layer:
    - skill routing
    - workflow logic
    - output contract
    - references or explanation if the mental model changes
- Common misreadings to avoid:
  - `dynatrace-investigation` is not a second canonical writer for the incident page.
  - `trial mode` is not partial mode or lightweight mode.
  - `incident-followup-planning` is not the right starting point for raw root-cause analysis from scratch.
  - a correlated deployment is not automatically the proven cause.
  - a downstream hot service is not automatically the owning service for the incident page.

# Related Docs

- Tutorial:
  - `tutorials/first-incident-investigation-trial-mode.md`
- How-to:
  - `workflows/pagerduty-incident-analysis.md`
  - `workflows/incident-followup-planning.md`
- Reference:
  - `references/incident-analysis-surfaces.md`
  - `templates/incident-analysis-page.md`
  - `templates/dynatrace-investigation-result.md`
