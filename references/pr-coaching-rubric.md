# PR Coaching Rubric

Use this reference when normalizing per-PR findings so the parent workflow can aggregate recurring patterns across multiple pull requests and multiple reviewer types.

## Core Rule

Normalize each issue into four parts:

- one `coaching tag`
- one short `symptom description`
- one `evidence status`
- one `reviewer source`

Keep the symptom concrete enough that the parent can tell whether two PRs are showing the same problem or only the same broad category.

## Coaching Tags

- `requirements-and-scope`
  The PR misses the story, accepts the wrong requirement premise, or widens scope without enough reason.
- `correctness`
  The implementation contains a logic bug, broken condition, invalid state transition, or data-handling defect.
- `failure-handling`
  The change lacks enough edge-case handling, null or empty handling, timeout handling, retry handling, or defensive checks.
- `tests-and-verification`
  The PR lacks focused proof, regression coverage, or verification depth for the risk it introduces.
- `interface-and-contract-safety`
  The change risks backward compatibility, schema safety, message compatibility, API contract safety, or migration safety.
- `observability-and-operability`
  The change lacks logs, metrics, tracing, rollout safety, alertability, or debuggability.
- `maintainability-and-clarity`
  The code is unnecessarily hard to read, overly coupled, duplicated, or brittle to change.
- `review-handling`
  The engineer repeatedly leaves valid review intent unresolved, responds without enough technical grounding, or needs repeated nudges on the same kind of issue.

When recording strengths, use the same tags when helpful so the parent can describe recurring positive habits too.

## Evidence Status

- `historical-code-supported`
  The issue is supported by the PR's own code state at review time.
- `current-code-supported`
  The issue is still visible in the current branch or default-branch code after the PR.
- `review-only`
  Reviewers raised it, but the code evidence is weaker or incomplete.
- `stale-or-disproven`
  The review theme does not hold up against the code that actually landed or the relevant historical context.

Prefer `historical-code-supported` for the main aggregation because this workflow is judging recurring PR behavior, not only the current repository state.

## Reviewer Source

- `human`
  The signal came primarily from human reviewers.
- `ai`
  The signal came primarily from AI reviewers or bots.
- `mixed`
  Both human and AI reviewers raised materially the same issue.
- `independent`
  The parent or child assessment found the issue directly without relying on reviewer comments.

## Recurrence Rules

- Treat a pattern as recurring only when the same concrete symptom appears in at least 2 PRs or at least 25 percent of the analyzed sample.
- Weight evidence in this order:
  - `current-code-supported`
  - `historical-code-supported`
  - `review-only`
- When evidence status is otherwise equal, prefer clusters supported by `mixed` or `independent` sources over clusters supported by one weak reviewer thread.
- Do not count `stale-or-disproven` themes toward recurrence.
- Keep one broad category split into multiple clusters when the symptoms differ in a way that changes the coaching advice.

## Coaching Rules

- Convert recurring issue clusters into habits, checks, or review prompts.
- Prefer coaching language such as `missing focused regression proof on risky branches` over vague labels such as `testing bad`.
- Preserve recurring strengths alongside recurring issues so the final report stays balanced.
