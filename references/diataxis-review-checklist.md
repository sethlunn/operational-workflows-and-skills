# Diataxis Review Checklist

Use this reference when reviewing a documentation change or when cleaning up an existing page.

This checklist works best section by section, not only file by file.

## Step 1: Identify The Reader Need

For the section you are reviewing, ask:

- Is the reader trying to learn by being guided?
- Is the reader trying to complete a task?
- Is the reader trying to look up exact facts?
- Is the reader trying to understand why something is shaped this way?

Choose the dominant need before judging the section.

## Step 2: Use The Compass

Ask two questions:

- Does this section inform action or cognition?
- Does it serve acquisition or application?

The result tells you the expected doc type:

- action + acquisition: tutorial
- action + application: how-to
- cognition + application: reference
- cognition + acquisition: explanation

## Step 3: Check For Mode Drift

Look for these common failures:

- procedural steps mixed with architectural rationale
- reference tables interrupted by narrative digressions
- tutorials that become option catalogs
- how-to guides that stop to teach background theory
- explanation that quietly contains required operational steps

If you see one of these, the section is probably in the wrong mode.

## Step 4: Decide The Fix

Choose one of these actions:

- keep it where it is because the mode is correct
- trim it so it stays in one mode
- split the mixed section into two docs or two labeled sections
- move the content to `tutorials/`, `workflows/`, `references/`, `templates/`, or `explanations/`
- replace generic structure with a better template

Prefer the smallest change that makes the reader need clearer.

## Step 5: Check Repo Layer Fit

Ask whether the content is in the right repo layer:

- `tutorials/` for guided first runs
- `workflows/` for task completion
- `references/` for stable lookup material and review rules
- `templates/` for output contracts and document skeletons
- `explanations/` for rationale, tradeoffs, and mental models

If the content is in the wrong layer, move it instead of only rewriting it.

## Step 6: Review Section Quality

For the chosen mode, check:

- Tutorial:
  - one guided path
  - explicit start and end state
  - recovery notes
- How-to:
  - clear task outcome
  - exact steps
  - validation and failure points
- Reference:
  - exact, scannable, stable
  - no hidden procedure
- Explanation:
  - clear why
  - explicit tradeoffs
  - links to action docs instead of replacing them

## Step 7: Make Mixed Artifacts Explicit

Some artifacts will still contain multiple modes, especially incident write-ups and landing pages.

When that happens:

- label the sections clearly
- keep each section internally consistent
- avoid switching mode mid-paragraph or mid-list

## Fast Questions

Use these when you want a quick pass:

- What is this section helping the reader do right now?
- Would a reader turn to this while working or while studying?
- Is this telling them what to do or why it is this way?
- If I removed this section from the current doc, where would it belong?

## Repo Example

Good use of the checklist:

- keep incident execution steps in `workflows/pagerduty-incident-analysis.md`
- keep the rationale for parent/child incident structure in `explanations/incident-analysis-pattern.md`

That split preserves a task-oriented workflow while still keeping the operating model documented.
