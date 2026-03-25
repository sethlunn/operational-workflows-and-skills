# Babysit PR

Review a GitHub pull request end to end: resolve the current review state, validate comments against the current diff, decide which comments are worth taking, and draft or post concise signed replies.

## Inputs

Accept any of:

- a PR number
- a GitHub PR URL
- a ticket-like phrase that can be resolved to a PR

## Workflow

1. Identify the PR.
- Accept a PR number or GitHub PR link directly.
- If the user gives only a ticket-like phrase, find the matching PR before proceeding.
- Read the PR body before taking a position on review comments.

2. Build the current comment state.
- Fetch inline review comments and replies.
- Prefer unresolved threads first.
- Separate comments into:
  - actionable and likely correct
  - questionable or invalid
  - already addressed or stale

3. Review the current code, not only the review snapshot.
- Inspect the current diff against the PR base branch.
- Verify whether each comment matches the current code path.
- Use targeted local verification when a comment claims a bug, regression, or missing coverage.

4. Decide how to handle each comment.
- If a comment is valid, explain the likely fix and ask the user before making code changes.
- If a comment is invalid, stale, or not worth taking, reply directly without asking first.
- Bias toward accommodating preferred reviewers when the request is safe and reasonable.

5. Reply on the PR.
- Reply directly under the relevant inline comment whenever possible.
- Keep replies concise, concrete, and professional.
- For valid comments after a fix, state what changed and what verification ran.
- For invalid or stale comments, explain why the comment does not apply to the current code.
- End every reply exactly as:
  - `sethlunn's pal, Thing 2`

6. Gate code changes behind explicit user approval.
- Do not edit code until the user approves.
- After approval, implement the change, run the smallest relevant verification, and then reply on the PR thread with the result.

## Preferred Reviewers

Be more accommodating to comments from:

- `davidsgbang`
- `TylerLidenZip`
- `rbtres22`

For these reviewers:

- default to treating suggestions and questions as good-faith signals worth satisfying
- make the requested change if it is safe and reasonable
- push back only when the request would introduce a bug, regression, materially worse design, or unnecessary complexity

## Command Patterns

Prefer these GitHub queries:

- `gh pr view <number> --json ...`
- `gh api repos/<owner>/<repo>/pulls/<number>/comments --paginate`
- GraphQL review-thread queries when you need resolved versus unresolved state

Prefer these local checks:

- `git diff origin/<base>...HEAD -- <path>`
- `rg` for locating symbols mentioned in comments
- targeted `dotnet test` or other narrow test runs when verification matters

## Decision Standard

A comment is invalid when at least one of these is true:

- the current code already addresses it
- the reviewer is commenting on outdated diff context
- the suggestion would introduce a bug or regression
- the suggestion is based on a false premise that can be verified locally

When in doubt:

- verify first
- ask before editing code
- reply only after the position is defensible from the current code
