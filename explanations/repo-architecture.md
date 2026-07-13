# Repo Architecture

This repo exists to keep operational logic portable across agent systems.

The main design choice is simple:

- put the real operating procedure in shared Markdown
- keep model-specific wrappers thin

That keeps the business logic in one place and reduces the chance that Codex, Claude Code, and a future workflow runner drift into different operating models.

## Why Shared Workflows

The files in `workflows/` are the canonical task procedures.

They are written to be readable by both humans and agents. That matters because operational work changes often, and the lowest-friction way to inspect or update the procedure is to edit one Markdown playbook rather than several model-specific prompt implementations.

## Why Thin Model Adapters

The files in `codex/*/SKILL.md` and `claude/*/SKILL.md` should stay thin on purpose.

Their job is to give each agent environment a native trigger and point it at the right shared workflow, reference, and template files. They should not become a second copy of the business logic. Once they start carrying real procedure, the repo loses its portability advantage.

Most current capabilities have both a Codex adapter and a Claude adapter. Adapter-specific metadata remains local to the adapter: Codex skills also carry `agents/openai.yaml`, while Claude discovers the `SKILL.md` folders linked under its home directory. A capability can be environment-specific when its operating contract requires it; the current `peer-review` entry is Claude-only. The workflow remains shared whenever the procedure itself is portable.

## Why Launchers And Links Are Separate From Workflows

The scripts under `scripts/` handle deterministic environment setup rather than business procedure.

- `link-*` installs repo-backed skill and shared-root symlinks.
- `sync-*` chooses the newest safe local or tracked-upstream source before relinking.
- `*-session` checks for native CLI updates, refreshes links, and starts the agent with this repo available.
- `*-incident-session` also performs harmless read-only tool preflight before waiting for a real incident id.

Keeping that behavior in scripts makes session setup repeatable while leaving the Markdown workflows readable and agent-independent.

## Why References Are Separate

The files in `references/` exist so workflows can stay focused on action.

A workflow should tell the reader what to do. It should not also carry every routing table, query shape, environment rule, or field-level reminder inline. Pulling those details into reference docs keeps task docs shorter and makes exact facts easier to maintain.

## Why Templates Are Separate

The files in `templates/` define output contracts.

They are not the workflow itself. They give investigators and agents a stable shape for parent incident pages, child investigation results, and similar artifacts. Keeping templates separate prevents task instructions from being buried inside output scaffolding.

## Why Incident Analysis Uses Parent And Child Investigations

The incident workflow is intentionally split between orchestration and bounded evidence gathering.

The parent workflow owns the overall incident narrative and, in publish mode, the parent Confluence page. Child investigations answer one narrow question at a time and return a structured evidence package.

That split exists for two reasons:

- it prevents each sub-investigation from trying to explain the whole incident
- it makes the evidence easier to audit, compare, and synthesize

## Practical Rule

When adding new material to this repo, decide which job the document is doing before writing it:

- teaching
- helping someone perform a task
- providing exact facts
- explaining why the system is shaped this way

If one document tries to do all four, it will usually be harder for both operators and agents to use correctly.
