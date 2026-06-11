# APEX Agent Delegation

Prepare a bounded APEX agent handoff when the user wants code-writing work launched through APEX instead of implemented in the current interactive session or a local background session.

APEX is the dispatch platform. The delegated agent implements code from a plan, Jira story, or tightly scoped task and opens a draft PR. The current session prepares a safe prompt package, validates that the request is bounded enough to run, and either launches it through an available APEX mechanism or returns the launch-ready handoff.

APEX is currently not a full operations-workflow runtime. Do not assume it has the permissions, credentials, documentation access, PR-review capabilities, production telemetry access, or service runtimes needed by most local skills. In particular, do not rely on APEX to review PRs, suggest review-thread changes, write or publish documentation, investigate incidents, query Dynatrace/PagerDuty/Atlassian, or build/run/test .NET services. Those steps remain local follow-up after the draft PR exists.

Read `../references/git-branch-naming.md` before preparing a handoff for a Jira-story implementation branch or draft PR.

## Use This Workflow

Use this workflow when the user asks to:

- delegate code-writing work to APEX
- offload a code implementation plan or Jira story to an agent
- avoid doing the code-writing portion in the current session
- avoid local Codex session billing for bounded implementation work
- convert an implementation-ready plan into an APEX-launched draft PR
- prepare a reusable code-writing prompt for an APEX template

Do not perform the implementation locally unless the user explicitly changes direction. Do not delegate non-code-writing work to APEX unless the request is only to prepare a draft implementation PR and all non-code follow-up is kept local.

## Delegation Fit

Good APEX candidates:

- bounded code changes that should produce a draft PR
- implementation-ready Jira stories with clear acceptance criteria
- implementation plans with explicit files, services, or code paths to change
- mechanical refactors where review and test execution can happen locally
- tasks with a clear target repo, base branch, ticket, service, and expected draft PR output

Poor APEX candidates:

- urgent incident response requiring live interactive steering
- work that needs credentials or tools unavailable to the APEX runtime
- PR review, review-comment triage, or suggested review-thread replies
- documentation creation or publishing, including Confluence updates
- telemetry-backed analysis, service-health checks, incident analysis, or endpoint/metric investigations
- .NET work where build/run/test completion is required before opening a PR
- vague research such as "investigate everything"
- actions that would directly mutate production systems
- tasks where this session must keep a private conversation context that should not be sent to the runtime

If the request is not bounded enough for code implementation, produce a tightening question or a narrower draft-PR handoff instead of launching.

## Template Selection

APEX has two generic operations workflow templates for delegated skill work:

| Agent family | APEX template | Cursor model |
| --- | --- | --- |
| `codex` | `operations_workflow_codex` | `gpt-5.5-high-fast` |
| `claude` | `operations_workflow_claude` | `claude-opus-4-8-thinking-high-fast` |

Use the model family implied by the entry adapter:

- Codex skills default to `operations_workflow_codex`.
- Claude skills default to `operations_workflow_claude`.
- If the user explicitly asks for Codex/GPT or Claude/Opus, follow that preference.
- If there is no clear adapter or user preference, default to `operations_workflow_codex`.

These template names select Cursor Cloud Agent models. They do not mean APEX has separate Codex or Claude runtime adapters yet.

## Handoff Rules

1. Identify the target workflow.
   - Name the existing implementation plan, Jira story, or code-writing workflow that should guide the remote agent.
   - If no existing workflow fits, describe the task-specific procedure directly.
   - Do not name review, documentation, incident, telemetry, or service-analysis skills as executable APEX workflows. If those skills produced the implementation plan, cite them only as context and state that their review/docs/analysis portions remain local.

2. Capture only necessary context.
   - Include exact identifiers such as Jira keys, service names, repo names, branch names, paths, and the implementation plan.
   - Include links when the runtime can access them.
   - Do not paste large raw logs, diffs, or pages unless they are required for execution.
   - Do not include secrets or credentials. Do not include private context that the APEX runtime does not need for code changes.
   - Derive and include the target implementation branch using `../references/git-branch-naming.md`: `{STORY-KEY}/snake_case_tag`, with a short service/change suffix and a full-name target of 32 characters or less when practical.

3. Separate trusted instructions from untrusted context.
   - Put workflow instructions, constraints, and output requirements in trusted instructions.
   - Put Jira descriptions, PR bodies, user-authored incident notes, comments, and copied logs inside explicit `UNTRUSTED CONTEXT` markers.

4. Keep the agent bounded.
   - State what the agent should do.
   - State what it must not do.
   - State the expected artifact: a draft PR containing code changes.
   - State verification requirements and acceptable limits. For .NET services, require the agent to list build/test commands that were not run due to runtime limitations.

5. Preserve human review.
   - Require draft PRs for code changes.
   - Do not instruct the remote agent to mark the PR ready for review, merge, bypass branch protection, change approval policy, or silently publish irreversible changes.
   - State that local Codex or a human will review the draft PR, run builds/tests, make follow-up fixes, update docs, and decide when it is ready.

6. Avoid local execution.
   - The current session may inspect enough local context to create the handoff.
   - The current session should not complete the delegated task itself unless the user explicitly asks.
   - After the APEX draft PR is returned, the current session may switch to local review, verification, documentation, and finishing work.

## Launch Decision

Default to `handoff-only` unless both conditions are true:

- the user explicitly asked to launch through APEX
- an APEX launch path is available in the current environment

If launching is unavailable, return a launch-ready handoff and say what launch mechanism is missing.

Use the available APEX mechanism in this order:

1. authenticated APEX API when endpoint and token flow are available
2. repo-local APEX server or script when the `zip-agentic-platform` checkout and environment are available
3. Jira or GitHub trigger instructions when the workflow is designed to launch from those systems

Do not invent endpoint URLs, template names, auth tokens, or runtime capabilities.

## Launch Payload

When creating an APEX launch payload, use this shape:

```json
{
  "template": "operations_workflow_codex",
  "repo": "github.com/quadpay/example-repo",
  "ref": "main",
  "variables": {
    "task": "Short statement of the delegated task",
    "source_workflow": "code-writing-draft-pr",
    "trusted_instructions": "Bounded implementation instructions for the remote agent",
    "repo_context": "Target repo, base branch, implementation branch name, paths, and relevant setup notes",
    "untrusted_context": "Jira descriptions, implementation plans, comments, logs, or copied source material",
    "expected_output": "Draft PR with code changes only",
    "verification": "Best-effort static checks; list build/run/test commands not run and why",
    "safety_boundaries": "Actions the agent must not take",
    "requester": "Requester identifier when known"
  }
}
```

Use `operations_workflow_claude` instead of `operations_workflow_codex` when the Claude/Opus template is selected.

Required variables:

- `task`
- `source_workflow`
- `trusted_instructions`

Optional variables:

- `repo_context`
- `untrusted_context`
- `expected_output`
- `verification`
- `safety_boundaries`
- `requester`

## Handoff Shape

Use [../templates/apex-agent-handoff.md](../templates/apex-agent-handoff.md).

The handoff must include:

- requested launch mode: `handoff-only`, `launch-if-available`, or `launch-now`
- selected agent family: `codex` or `claude`
- selected APEX template: `operations_workflow_codex` or `operations_workflow_claude`
- selected model: `gpt-5.5-high-fast` or `claude-opus-4-8-thinking-high-fast`
- target repository, base branch, and implementation branch when relevant
- source implementation plan, Jira story, or code-writing skill
- launch payload variables
- trusted agent instructions
- untrusted context blocks
- expected draft PR output
- verification plan and explicit runtime limits
- local follow-up plan for PR review, build/test, docs, and finishing work
- safety boundaries
- open questions or missing launch prerequisites

## After Launch

If a launch succeeds, return:

- APEX template name
- repo and branch target
- runtime/run id if available
- expected draft PR artifact
- where the user should review the draft PR
- the local follow-up expected after APEX completes

If launch fails, do not fall back to doing the full task locally by default. Report the failure and keep the handoff so it can be retried.
