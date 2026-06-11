# Git Branch Naming

Use this reference when a skill creates or delegates a Git branch or draft PR for implementing a Jira story.

## Jira Story Branches

Use this branch shape:

```text
{STORY-KEY}/snake_case_tag
```

Rules:

- Use the Jira key as the prefix exactly as the story key appears, uppercase, such as `DQ-9143`.
- Use a lowercase snake_case suffix.
- Keep the full branch name at or under 32 characters when practical, including the Jira key and slash.
- Prefer short, specific tags over full story-title slugs.
- Include the short service, component, or package name plus the main change.
- Avoid generic suffixes such as `fix`, `changes`, `update`, `story`, or `implementation`.
- Shorten obvious words before exceeding the limit, for example `auditlogs_net10` instead of `upgrade_auditlogs_to_net10`.
- If a branch name collides, add the shortest useful disambiguator while staying within the limit.

Examples:

```text
DQ-9143/auditlogs_net10
DQ-9143/auditlogs_efcore
DQ-9172/risk_rules_cap
PAY-204/clc_retry_backoff
```
