# Confluence Analysis Writing Standard

Use this reference when a workflow produces an analysis page backed by code inspection and telemetry evidence.

Use `references/diataxis-writing-rules.md` first when the page type itself is still unclear.

## Writing Rules

- Use concise, decision-oriented prose.
- Use exact absolute dates and timestamps.
- State the exact scope and exact filters used.
- Put important caveats in the page body, not only in the chat response.
- Include the exact query shapes or DQL used for the core evidence when telemetry is central to the conclusion.

## Evidence Discipline

Separate these clearly in the page:

- direct evidence from code
- direct evidence from telemetry
- interpretation or inference

Do not blur "the code emits this" with "the telemetry likely means this."

## Caveat Placement

Always state important limitations in a visible section, usually:

- executive summary
- methodology
- limitations and caveats

Typical limitations to call out:

- event counts vs distinct-entity limits
- dimension coverage gaps
- fixed-window source-of-truth choice
- partial windows
- environment restrictions

## Page Content Standard

Most analysis pages should include:

- executive summary
- exact scope and methodology
- code-path explanation
- telemetry or query evidence summary
- trend or breakdown section if requested
- limitations and caveats
- exact query shapes used

## Canonical Writer Rule

If multiple agents or sub-investigations contribute evidence:

- define one canonical writer for the final page
- child investigations should return bounded evidence rather than trying to narrate the whole document
