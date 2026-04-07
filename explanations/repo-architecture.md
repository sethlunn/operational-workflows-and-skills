# Repo Architecture

This repo exists to keep operational logic portable across agent systems.

The main design choice is simple:

- put the real operating procedure in shared Markdown
- keep model-specific wrappers thin

That keeps the business logic in one place and reduces the chance that Codex, another agent, and a future workflow runner drift into different operating models.

## Why Shared Workflows

The files in `workflows/` are the canonical task procedures.

They are written to be readable by both humans and agents. That matters because operational work changes often, and the lowest-friction way to inspect or update the procedure is to edit one Markdown playbook rather than several model-specific prompt implementations.

## Why Thin Skill Adapters

The files in `codex/*/SKILL.md` should stay thin on purpose.

Their job is to point Codex at the right shared workflow, reference, and template files. They should not become a second copy of the business logic. Once they start carrying real procedure, the repo loses its portability advantage.

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
