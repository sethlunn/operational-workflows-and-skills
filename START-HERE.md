# Start Here

This repo is being organized around Diataxis:

- tutorials teach through a guided first run
- how-to guides help you complete an operational task
- reference gives exact facts and contracts
- explanation gives rationale and mental models

Use the section that matches your need right now.

## If You Need A Guided First Run

Go to `tutorials/`.

Start with:

- First incident investigation in `trial mode`:
  `tutorials/first-incident-investigation-trial-mode.md`

## If You Need To Do Work Now

Go to `workflows/`.

Recommended starting points:

- PagerDuty incident to Confluence write-up:
  `workflows/pagerduty-incident-analysis.md`
- Dynatrace investigation router:
  `workflows/dynatrace-investigation.md`
- PagerDuty-owned service health rollup:
  `workflows/pagerduty-assigned-service-health.md`
- Service endpoint inventory and traffic analysis:
  `workflows/service-endpoint-traffic-analysis.md`
- Service metric and telemetry analysis:
  `workflows/service-metric-analysis.md`
- Incident follow-up planning:
  `workflows/incident-followup-planning.md`
- PR review triage:
  `workflows/babysit-pr.md`
- Independent PR review:
  `workflows/review-pr.md`

## If You Need Exact Facts

Go to `references/`, `templates/`, or `scripts/`.

Useful anchors:

- Diataxis authoring rules:
  `references/diataxis-writing-rules.md`
- Diataxis review checklist:
  `references/diataxis-review-checklist.md`
- Confluence routing:
  `references/confluence-routing.md`
- Dynatrace query shapes:
  `references/dynatrace-query-patterns.md`
- Fast-path triage rules:
  `references/dynatrace-fast-path.md`
- Subagent usage rules:
  `references/subagent-usage.md`
- Parent incident page contract:
  `templates/incident-analysis-page.md`
- Child investigation result contract:
  `templates/dynatrace-investigation-result.md`
- Non-incident child result contract:
  `templates/analysis-child-result.md`
- Generic tutorial template:
  `templates/tutorial-page.md`
- Generic how-to template:
  `templates/how-to-page.md`
- Generic reference template:
  `templates/reference-page.md`
- Generic explanation template:
  `templates/explanation-page.md`

## If You Need To Understand The System

Start with `explanations/repo-architecture.md`.

Then read `explanations/incident-analysis-pattern.md` if you want the rationale behind the parent-and-child incident workflow.
Read `explanations/dynatrace-investigation-pattern.md` if you want the rationale behind the Dynatrace router and branch model.

That explains:

- why the workflows are shared Markdown instead of model-specific prompts
- why the Codex skill wrappers stay thin
- why incident analysis uses a parent-and-child investigation model
- why references and templates are kept separate from workflows

## If You Are New To The Repo

Use this order:

1. Read `README.md`.
2. Read `explanations/repo-architecture.md`.
3. Pick one workflow in `workflows/`.
4. Open the supporting template or reference docs only when the workflow tells you to.

## Current Gap

The repo is strongest in how-to guides and reference. It now has its first true tutorial, but the tutorial quadrant is still sparse.

The next useful tutorial additions are:

- first shared workflow plus thin Codex adapter
- first publish-mode incident write-up
