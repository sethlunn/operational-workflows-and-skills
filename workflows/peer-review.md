# Peer Review

Draft Zip FY26 performance peer reviews about one or more colleagues: build a bounded evidence pack per peer from their delivered Jira work and their GitHub code-review history, fold in the AI-usage and collaboration observations only the reviewer holds, then write the two required Lattice entries: 2-3 value-anchored strengths and 1-2 growth opportunities, each backed by a real example and its impact.

## Inputs

Accept any of:

- a roster of peers to review (names, ideally with GitHub handles and Jira identities)
- a single peer to draft now
- a request to write FY26 peer reviews, performance peer feedback, or Lattice peer entries for coworkers

Per peer, gather or accept:

- GitHub handle and the repo or org scope where they author
- Jira identity (display name or accountId) and the projects where they deliver
- the reviewer's relationship to them (same team, cross-team, how often they worked together)
- AI-usage notes and personal observations the reviewer supplies
- any peer-specific Jira keys or PR numbers the reviewer already has in mind

Optional inputs:

- FY26 window override
- PR sample size or time window
- whether to draft all peers now or one at a time
- the private output directory

Read `../references/peer-review-rubric.md` before shaping any claim.
Read `../workflows/review-pr.md` before reading an individual PR and its threads.
Read `../workflows/pr-author-coaching.md` before sampling several PRs by one author.
Read `../references/subagent-usage.md` before splitting evidence gathering across peers.
Read `../templates/peer-review-entry.md` for the per-peer output contract.

## Execution Posture

Default to `draft-only, reviewer-verified`.

- Write drafts to a private path outside this repo. Default to `~/peer-reviews-fy26/`, one file per peer. Never write peer feedback into the tracked repo.
- Never submit to Lattice and never message the peer. The reviewer pastes the two answers in themselves after verifying.
- The AI-usage notes and the reviewer's observations come only from the reviewer. Do not infer them from code or tickets, and do not invent examples, metrics, or quotes.
- Claim only what the evidence supports. Surface thin claims to the reviewer and ask for a concrete example instead of filling the gap.

## Defaults

- FY26 window: Zip runs an Australian fiscal year, so default to `2025-07-01` through `2026-06-30`. Confirm the exact window with the reviewer if delivery dates sit near the boundary.
- Jira scope: the peer's epics, features, and stories that landed in FY26, weighted to substantive delivery and ownership.
- GitHub scope: prefer PRs the peer authored that the reviewer actually reviewed, since those carry first-hand collaboration signal; otherwise the most recent substantive non-draft merged PRs in the named repo.
- Exclude low-signal work: bot PRs, dependency bumps, release chores, trivial tickets, unless the reviewer wants them.
- If a peer's evidence is too thin for a confident growth point, say so rather than manufacturing a weakness.

## Subagent Posture

- Optional fit. Strong only at roster scale (for example 10 peers).
- Good split: one bounded evidence-gathering child per peer, returning a compact evidence pack, never a finished review.
- Keep the parent as the canonical writer and the single thread that talks to the reviewer.
- Do not delegate the AI-usage or observation inputs; those are collected from the reviewer in the parent thread.
- When used, children should return the `Evidence Pack` portion of `../templates/peer-review-entry.md`, with Jira and GitHub signals separated.

## Workflow

1. Build the roster and per-peer scope.
- List each peer with: GitHub handle, Jira identity, team, and relationship to the reviewer.
- Resolve Jira accountIds early with `lookupJiraAccountId` so JQL is exact.
- Confirm the GitHub repo or org scope. Reuse the named repo, or the current checkout if it is the intended codebase.
- Confirm the FY26 window.

2. Gather delivered work from Jira.
- Find the peer's substantive FY26 delivery: epics, features, and stories they owned or drove.
- Capture for each: key, type, title, status, and the reviewer's relationship to it.
- Prefer items that show scope, ownership, prioritisation, follow-through, and customer-facing outcomes.

3. Gather code-review signals from GitHub.
- Build a bounded PR sample per peer using the `pr-author-coaching` sampling discipline.
- For each PR capture: title, scope, quality and risk-handling signals, and the collaboration signal from review threads (responsiveness, how they took feedback, how they enabled reviewers).
- Read threads with the `review-pr` discipline; treat comments as signals, not verdicts.
- Keep human-review, AI-review, and code-supported signals separate.

4. Collect the reviewer-only inputs.
- Ask the reviewer, per peer, for:
  - how the peer used AI or automation to achieve the work (which tool or approach, what they built or accelerated, the observed outcome)
  - the reviewer's own interactions and observations on how the peer uses AI to empower their development process
  - any collaboration moments only the reviewer witnessed
- Do not proceed to a polished AI-impact claim without a concrete instance from the reviewer.

5. Map evidence to values and the two entries.
- Using `../references/peer-review-rubric.md`, normalize each candidate claim into value, behavior, impact, and evidence source.
- Cluster into 2-3 strengths and 1-2 growth opportunities.
- Write each as Situation -> Behavior -> Impact, anchored to a Zip value, with a metric where one exists.
- Frame each growth point as a way to increase impact, with one actionable next step. Never as a character flaw.
- Weight Jira- and code-supported evidence above impressions; mark thin claims for reviewer confirmation.

6. Draft each entry.
- Use `../templates/peer-review-entry.md`. Keep the evidence pack and the paste-ready answers clearly separate.
- Write to the private output directory, one file per peer.
- Keep the two answers concise: a few tight sentences per example.

7. Run the reviewer verification loop.
- Present each draft, calling out any low-confidence claims and any place a concrete example is still missing.
- Apply the reviewer's corrections. Re-check against the pre-submit checklist in the template.
- Only the reviewer decides a draft is final and pastes it into Lattice.

8. Track progress across the roster.
- Keep a per-peer status: evidence gathered, reviewer inputs collected, drafted, verified.
- Respect the cycle deadline (FY26 peer reviews are due EOD Friday June 26).

## Command Patterns

Jira via the Atlassian MCP:

- `lookupJiraAccountId` to resolve a peer's name to an accountId.
- `searchJiraIssuesUsingJql` with JQL such as:
  - `assignee = "<accountId>" AND statusCategory = Done AND resolutiondate >= "2025-07-01" AND resolutiondate <= "2026-06-30" ORDER BY resolutiondate DESC`
  - add `AND issuetype in (Epic, Story, Task)` or `AND project = <KEY>` to focus the sample.
- `getJiraIssue` for the detail and parent epic of a specific key.
- The Atlassian MCP intermittently returns a client-serialization 400. Do not reformat the request, just retry.

GitHub via `gh`:

- `gh search prs --author <handle> --repo <owner>/<repo> --state merged --limit <n> --json number,title,url,state,createdAt,mergedAt,isDraft,repository`
- `gh pr view <number> --repo <owner>/<repo> --json number,title,body,url,author,baseRefName,headRefName,headRefOid,files,reviewDecision`
- `gh api repos/<owner>/<repo>/pulls/<number>/comments --paginate`
- `gh api repos/<owner>/<repo>/pulls/<number>/reviews --paginate`
- `gh pr diff <number> --repo <owner>/<repo>` for the change itself.

## Output Rules

- Two entries per peer, matching the Lattice form: greatest strengths (2-3), and where could they grow to increase their impact (1-2).
- Every claim is objective, specific, value-anchored, and written as Situation -> Behavior -> Impact.
- Use exact identifiers: Jira keys, PR numbers, repos, and dates in the evidence pack.
- Growth is framed as an opportunity with a next step, never as a character judgment.
- No fabricated examples, metrics, or AI usage. Thin claims are confirmed with the reviewer first.
- Draft only, to the private path. Never submit to Lattice and never contact the peer.
