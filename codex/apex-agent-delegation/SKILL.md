---
name: apex-agent-delegation
description: "Prepare and hand off bounded code-writing prompts to an APEX-launched agent that can implement from a plan or Jira story and open a draft PR. Use when the user asks to delegate implementation work through APEX, but keep review, documentation, runtime verification, and finishing work local."
---

# APEX Agent Delegation

Read [../../workflows/apex-agent-delegation.md](../../workflows/apex-agent-delegation.md) before taking action.

Use this skill to package code implementation work for APEX, not to perform the implementation locally.

Current APEX capability boundary: use APEX only for bounded code writing that should produce a draft PR. Do not use APEX for PR reviews, review-comment triage, documentation publishing, Confluence/Jira updates, telemetry investigation, incident analysis, or final verification. APEX currently cannot be assumed to have the permissions, credentials, documentation access, or service runtimes needed for those workflows.

For .NET service work, assume APEX cannot build, run, or test the service. The handoff must require the APEX agent to state which build/test commands were not run and why, and must leave local review, runtime verification, documentation, and final PR readiness to this Codex session or a human.

For Jira story implementation handoffs, include an implementation branch name that follows `{STORY-KEY}/snake_case_tag`; use a short service/change suffix and keep the full branch name at or under 32 characters when practical.

Default to the `operations_workflow_codex` APEX template unless the user explicitly asks for the Claude/Opus template.

Default to a handoff draft unless the user explicitly asks to launch and the APEX launch mechanism is available in the current environment.

When preparing the handoff, preserve the original user goal, exact identifiers, required repo context, safety boundaries, expected draft PR output, known verification limits, and the required local follow-up plan. Use [../../templates/apex-agent-handoff.md](../../templates/apex-agent-handoff.md) as the output contract.
