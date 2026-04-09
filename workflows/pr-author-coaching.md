# PR Author Coaching

Coach one or more GitHub engineers using their recent pull-request history: build a bounded sample of recent authored PRs, assess each PR's historical review signals and code quality, then synthesize recurring strengths and recurring problem patterns into actionable guidance.

## Inputs

Accept any of:

- one or more GitHub usernames
- a GitHub profile URL plus repo or org scope
- a request to analyze an engineer's recent PR history for recurring review or code-quality issues

Optional inputs:

- repo or org scope
- PR count
- time window
- whether to include open PRs
- whether to produce only an in-session report or also create a local draft

Read `../workflows/review-pr.md` before assessing any individual PR.
Read `../references/subagent-usage.md` before splitting the work into bounded child assessments.
Read `../references/pr-coaching-rubric.md` before tagging recurring issues.
Read `../templates/analysis-child-result.md` when using one child assessment per PR.

## Execution Posture

Default to `session-only coaching`.

In the default mode:

- do not comment on PRs
- do not modify code
- do not contact the engineer directly
- return a coaching-ready report in session

Only create a local draft or external document if the user explicitly asks for that follow-up.

## Sampling Defaults

- Default repo scope to the repo explicitly named by the user.
- If no repo is named and the current working directory is already a clear git checkout for the intended codebase, use that repo.
- Default to the 5 most recent substantive non-draft merged PRs per user from the last 180 days.
- Exclude obvious bot PRs, release chores, dependency bumps, and other low-signal mechanical work unless the user explicitly wants them included.
- If fewer than 3 substantive PRs remain for a user, widen the window or say plainly that the confidence is low.
- Include open PRs only when the user explicitly asks for active-work analysis or when the sample would otherwise be too small.

## Subagent Posture

- Optional fit, not the default.
- Stay local when the request covers one engineer and only 1 or 2 PRs.
- Consider one fresh child session per PR only when the sample is large enough that the parent would otherwise get context-heavy.
- When fresh child sessions are not available, use 2 to 4 bounded child agents in parallel.
- Keep the parent as the canonical writer for the final report.
- Prefer one PR per child assessment.
- Do not split one PR into overlapping code and review children unless the PR is unusually large and the scopes can be kept disjoint.
- Each child should return `../templates/analysis-child-result.md`.

## Workflow

1. Resolve the exact analysis scope.
- Write down:
  - GitHub usernames
  - repo or org scope
  - exact PR count or time window
  - whether open PRs are included
- Prefer one repo or one tightly related repo set.
- If the request spans very different repos, warn that cross-repo aggregation lowers confidence because reviewer norms and codebase constraints differ.

2. Build the candidate PR set.
- Use GitHub search to gather recent authored PRs for each target user.
- Record the exact candidate list before filtering:
  - PR number
  - repo
  - title
  - URL
  - state
  - created date
  - merged date when present
- Filter out low-signal PRs explicitly and keep the excluded list in the final scope section.

3. Decide the orchestration strategy.
- Stay local when the total sample is 1 or 2 PRs and coherence matters more than parallelism.
- Split into one child assessment per PR when the sample is larger.
- If multiple users are being analyzed and each has only 1 or 2 PRs, one child per user is acceptable.
- Give each child one exact question:
  - what issues surfaced in review and code quality for this PR
  - which patterns are corroborated by code at PR time
  - which patterns are only reviewer noise or stale context

4. Run one bounded assessment per PR.
- For each PR, gather:
  - title
  - body
  - author
  - base branch
  - head branch
  - head SHA
  - merge commit SHA when present
  - changed files
  - linked ticket references when present
- Read inline review comments, replies, top-level reviews, and resolved or unresolved thread state when needed.
- Treat review comments as signals, not as the source of truth.
- For merged or closed PRs, inspect the code at the PR's historical head SHA or merge commit when possible.
- Do not judge historical review comments only against today's default branch.
- If the user also wants to know whether a pattern still exists now, run that as a separate check and label it `current-state follow-up`, not as the historical PR judgment.
- Build a compact context pack using the same discipline as `../workflows/review-pr.md`, but keep the judgment anchored to the historical PR state.
- Normalize the strongest issues and strengths using `../references/pr-coaching-rubric.md`.

5. Require a strict child result contract.
- Each child result must include:
  - exact PR assessed
  - exact scope searched
  - exact historical revision or fallback evidence surface used
  - strongest human-review themes
  - strongest AI-review themes
  - issues corroborated by historical code
  - issues not corroborated or stale
  - normalized coaching tags
  - parent-ready summary
- When using `../templates/analysis-child-result.md`, put the review signals under `Evidence` and the normalized tags plus coaching interpretation under `Conclusion`.
- Reject child summaries that do not separate review signals from code-supported findings.

6. Aggregate recurring patterns in the parent.
- Cluster issues by:
  - normalized coaching tag
  - concrete symptom
  - reviewer-source mix
- Count a pattern as recurring when it appears in at least 2 PRs or in at least 25 percent of the analyzed sample.
- Weight direct code-supported evidence above raw reviewer frequency.
- Keep these separate:
  - corroborated recurring problems
  - review-only themes not strongly supported by code
  - stale or disproven comments
  - recurring strengths worth preserving
- When multiple engineers are included, preserve both:
  - per-user pattern clusters
  - cross-user clusters when they are genuinely similar

7. Turn the clusters into coaching guidance.
- For each high-confidence recurring problem:
  - describe the pattern in neutral engineering terms
  - cite the representative PRs
  - explain the likely failure mode or design habit behind it
  - give one short pre-PR check or working habit that would prevent it
- Keep the focus on reusable habits rather than blame.
- Include strengths so the coaching guidance does not flatten everything into defects.

8. Produce the final report.
- Include:
  - exact scope and confidence
  - included and excluded PRs
  - recurring strengths
  - recurring problem clusters
  - per-user summary when applicable
  - a short coaching checklist
  - a short re-measurement plan for the next sample

9. Optionally create a draft artifact.
- Only if the user explicitly asks:
  - create a local markdown draft
  - or prepare a page-ready write-up for later publishing
- Keep the parent report as the canonical artifact.

## Command Patterns

Prefer these GitHub queries:

- `gh search prs --author <user> --repo <owner>/<repo> --state merged --limit <n> --json number,title,url,state,createdAt,updatedAt,mergedAt,isDraft,repository`
- `gh pr view <number> --repo <owner>/<repo> --json number,title,body,url,author,baseRefName,headRefName,headRefOid,mergeCommit,state,reviewDecision,files`
- `gh api repos/<owner>/<repo>/pulls/<number>/comments --paginate`
- `gh api repos/<owner>/<repo>/pulls/<number>/reviews --paginate`
- GraphQL review-thread queries when resolved versus unresolved state matters

Prefer these code-history checks:

- `git diff <base-sha>...<head-sha> -- <path>`
- `git show <sha>:<path>`
- `gh pr diff <number> --repo <owner>/<repo>`
- `gh api repos/<owner>/<repo>/pulls/<number>/files --paginate`

## Output Rules

- Use exact usernames, exact repos, exact PR numbers, and exact dates.
- Separate:
  - human-review signals
  - AI-review signals
  - code-supported findings
  - stale or disproven review themes
- Say plainly when the sample is too small or too mechanically biased to support a strong coaching conclusion.
- Keep the output coaching-oriented and evidence-backed.
- Do not overfit one noisy reviewer comment into a recurring pattern without corroboration.
