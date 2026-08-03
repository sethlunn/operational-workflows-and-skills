---
name: implement-jira-story
description: "Implement a Jira story in a local repository from an issue key or Jira URL. Use when the user asks Codex to implement, build, code, or complete a Jira story: read the full issue including acceptance criteria and comments, resolve the target repository, prepare the latest master safely with a feature branch or worktree, make and verify the code changes, then commit, push, and open a draft pull request. Do not use for implementation delegation through APEX or for reviewing an existing pull request."
---

# Implement Jira Story

Read [../../workflows/implement-jira-story.md](../../workflows/implement-jira-story.md) before taking action.

Follow the shared workflow:

- read the full Jira issue and every comment before coding
- turn the issue fields and comments into one effective implementation contract
- preserve existing repository work by using a dedicated worktree when the current checkout is not safe to repurpose
- start from the latest `master` and use the `{STORY-KEY}/brief_snake_case_title` branch convention
- obey repository-local agent instructions, implement the smallest complete change, and verify it against the acceptance criteria
- commit only the verified story changes, push the feature branch, and open or update its draft pull request by default
- do not mark the pull request ready, merge it, transition Jira, edit the issue, or add Jira comments unless the user asks
