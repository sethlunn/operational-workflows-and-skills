# Local Context Layout

Use this reference when a skill creates a worktree, writes a non-repo local file, or looks for a local context or handoff file.

## Workspace Root

The workspace root is the directory that holds the git repo checkouts. On Seth's machine it is `~/zipRepos`.

The workspace root contains only:

- git repo checkouts, such as `quadpay-services`, `Platform`, `zip-agentic-platform`, and this repo
- worktrees for stories whose PRs are still open, named `<repo>-<lowercased story key>`, such as `quadpay-services-dq-9457`
- one `local-context/` directory for everything that is not a git checkout

Never write a loose file or a new non-repo directory at the workspace root. Route it into `local-context/` instead.

## local-context Directory Map

| Directory | Contents |
|---|---|
| `local-context/story-context/` | Per-story or per-epic working notes, breakdown context files, and scoping inputs |
| `local-context/docs/` | Local design and working docs not tracked in a repo, such as `camunda-decisioning-mock/` |
| `local-context/handoffs/` | Session and agent handoff files, including APEX delegation handoffs saved to disk |
| `local-context/incidents/` | Local incident notes and RAC drafts kept outside Confluence |
| `local-context/demos-and-presentations/` | Demo scripts, transcripts, decks, and supporting assets |
| `local-context/peer-reviews-fy26/` | Private peer-review drafts; never tracked in any repo |

Create a matching new purpose directory under `local-context/` only when none of the existing ones fits.

## Worktree Rules

- Create story worktrees as siblings of the main checkout at the workspace root: `<workspace-root>/<repo>-<lowercased story key>`.
- One worktree per story; branch from freshly fetched `origin/master` per the implement-jira-story workflow.
- After a story's PR merges, the worktree and its local branch are stale: remove the worktree with `git worktree remove` and delete the local branch. Do this only when the checkout is clean and the PR state is `MERGED`, and only when the user asks for cleanup or has pre-authorized it.
- Never leave worktrees for merged PRs accumulating at the workspace root.

## Rules For Skills

- Skills that read local context or handoff files should look in `local-context/story-context/` and `local-context/handoffs/` first when the user gives a bare filename.
- Skills that write local, non-repo artifacts must target the matching `local-context/` subdirectory, never the workspace root and never a tracked repo.
- Repo-tracked artifacts, such as `reviews/service-docs/` source sets in this repo, are unaffected by this layout; it governs only non-repo local files.
