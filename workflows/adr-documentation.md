# ADR Documentation

Create, review, revise, supersede, or repair drift in an Architecture Decision Record (ADR) while preserving its role as a durable record of one consequential architectural choice.

Before every ADR task, read:

- `../references/adr-writing-rules.md` as the authoritative source for QD format, content budgets, lifecycle policy, drift rules, and consistency checks
- `../templates/architecture-decision-record.md` as the required house-format skeleton for new records and structural repairs

When publishing to or updating Confluence, also inspect the current page, its lifecycle metadata, and useful version history before drafting an edit.

## Defaults And Boundaries

- Default to `draft mode`: return findings, proposed wording, or a local artifact without changing Confluence.
- Enter `publish mode` only when the user explicitly asks to create or update a Confluence ADR.
- Keep one independently reversible architectural choice per ADR.
- Use an ADR to preserve the decision, its stable drivers, alternatives, outcome, consequences, and confirmation conditions.
- Use `../workflows/technical-design-documentation.md` when the needed artifact primarily explains future-state architecture, interfaces, schemas, detailed flows, permission or package contracts, delivery sequencing, migration, rollout, or implementation mechanics.
- Link a companion technical design, runbook, rollout guide, evidence page, or Jira plan instead of embedding that living detail in the ADR.
- Never silently invent a decision invariant, stakeholder view, fact, or historical rationale. Surface unresolved questions and keep decision-blocking unknowns consistent with `Draft` or `Proposed` status.

## Resolve The Job

Classify the request before editing:

1. Draft a new ADR.
2. Review readiness or structural quality without rewriting.
3. Revise a `Draft` or `Proposed` ADR.
4. Audit and repair ADR drift.
5. Make a safe, meaning-preserving edit to an `Accepted` ADR.
6. Create a successor and supersede an earlier ADR.

Resolve the output mode separately: in-session response, local markdown, or explicit Confluence publication. For a review, present findings before proposed replacement wording.

## Establish Evidence And The Decision Invariant

Gather the strongest available sources: the current ADR and useful version history, linked Jira or initiative, companion designs, code or operational evidence, and stakeholder decisions. Distinguish confirmed facts from assumptions and time-sensitive evidence.

Before revising an existing ADR, capture:

- exact decision question
- lifecycle status
- exact selected or proposed option
- top drivers
- scope boundary
- decision owner

If the invariant cannot be recovered confidently, stop at an audit with targeted questions. Do not clean up prose in a way that fabricates or changes the decision.

## Draft A New ADR

1. Test whether the request contains exactly one consequential, independently reversible architectural choice. Propose a split when it does not.
2. Give the ADR a declarative decision title such as `Use X for Y` or `X as Y`.
3. Fill the lifecycle metadata in the template. Use `Draft` or `Proposed` until the decision is actually accepted.
4. Write the canonical sections in exact template order and within the authoritative content and size budgets.
5. Compare two to four peer alternatives, including the status quo when viable. Keep option names identical everywhere.
6. Use `Proposed option:` for `Draft` or `Proposed` records and `Chosen option:` only when the status and evidence support an accepted decision.
7. Put only observable decision checks in Confirmation; move task lists and implementation validation to companion artifacts.
8. Run the full consistency and drift audit in the authoritative rules.

## Review Readiness

Review without changing the artifact unless rewriting is requested. Report findings in this order:

1. decision clarity and whether the ADR contains exactly one decision
2. lifecycle and outcome consistency
3. canonical structure and exact option-name consistency
4. neutrality and strength of drivers and comparisons
5. consequence and confirmation quality
6. unsupported, contradictory, stale, or time-sensitive claims
7. material that should move to a named companion-document type
8. size, repetition, and split risk
9. decision-blocking questions and required reviewers

Conclude with a readiness judgment: `ready`, `ready after minor edits`, or `not ready`, and state the specific gates for progressing from Draft or Proposed to Accepted.

## Revise A Draft Or Proposed ADR

- Preserve the decision invariant unless the user intentionally changes the decision question and agrees to split or re-scope the record.
- Rewrite canonical sections in place. Replace stale detail and contradictions instead of appending a second interpretation or dated refinement diary.
- Keep the status `Proposed` while a decision-blocking assumption remains unresolved.
- Re-run the consistency audit after revision and identify stakeholders who still need to review before acceptance.
- Summarize substantive changes as retained, moved, removed, and clarified.

## Repair Drift

1. Inventory headings, approximate word count, repeated claims, contradictions, time-sensitive facts, and embedded non-ADR artifacts.
2. Compare the artifact against the canonical order and size guardrails. Trigger a split review under the conditions defined in the authoritative rules.
3. Classify each useful statement into one canonical ADR section or a named companion destination.
4. Produce a `retain / move / remove` plan and get user approval before a material rewrite or external mutation.
5. For `Draft` or `Proposed`, rewrite in place after approval. Delete duplicated, superseded, or non-decision prose instead of preserving it for history.
6. For `Accepted`, stop and propose a successor whenever repair would alter the decision invariant.
7. Run the consistency audit and report the before/after word count, section shape, extracted destinations, and unresolved evidence gaps.

## Edit An Accepted ADR Safely

Allow only spelling and formatting corrections, link repair, clearer wording that does not change meaning, lifecycle-link maintenance, and updated confirmation evidence. Preserve the historical body of `Rejected`, `Superseded`, and `Deprecated` records under the same rule.

Do not edit an Accepted ADR in place if the requested change affects its question, drivers, scope, selected option, or material consequences. Route that work to a successor ADR.

## Create A Successor And Supersede

1. Draft the new decision as a complete ADR with its own evidence and current date; do not merely patch the old record.
2. Add `Supersedes` to the successor and `Superseded by` to the earlier record using reciprocal links.
3. Mark the earlier record `Superseded` only when the successor decision is accepted. Until then, keep the successor `Draft` or `Proposed` and do not imply that the historical decision has changed.
4. Preserve the earlier body. Update only its lifecycle metadata and reciprocal link.
5. Verify that both records state compatible lifecycle facts and that the successor explains why new evidence requires a changed choice.

## Confluence Publication Safeguards

Apply these only in explicitly requested `publish mode`:

1. Confirm whether the operation is a new page or an update, and resolve the exact space, parent folder, page URL, or page id. Do not rely on title search when an existing page id or URL is available.
2. Read the latest page body, lifecycle metadata, version, and useful version history immediately before updating. Do not overwrite newer changes blindly.
3. Show or summarize the material edit and lifecycle effect before a rewrite, accepted-record correction, or supersession mutation. A prior request to review is not authorization to publish.
4. Preserve page identity and history when revising an existing ADR. Do not create a duplicate page as an accidental workaround.
5. Use a meaningful Confluence version message that describes the substantive decision-record change.
6. After publishing, fetch and inspect the rendered page for heading order, metadata, typed bullets, macros, tables, and links.
7. For supersession, verify both reciprocal links and statuses after both writes. If one write fails, report the partial state immediately; do not claim completion.
8. Report the page link, resulting version, version message, lifecycle state, and post-publication verification.

## Final Quality Gate

Run every item in the authoritative consistency and drift audit. Also verify that the artifact uses the template's exact canonical section order, stays focused on one decision, keeps option names identical, and contains no detailed-design material that belongs in a companion document.
