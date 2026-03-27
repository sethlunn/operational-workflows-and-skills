# Incident Investigation Lessons 2026-03-27

Use this note as rationale for the workflow guardrails added after the `2026-03-26` to `2026-03-27` Shopping and customer-portal incident investigations.

This is not the primary workflow. It is a compact record of the concrete cases that motivated the current guidance.

## Lessons

- Confluence folder ids are not page ids.
- Example: Shopping BE folder `5289574404` returned `404` when treated as a page id, but worked correctly as a Confluence page `parentId` with resulting `parentType = folder`.
- Implication: routing docs should treat mapped ids as folder parents, not page ids to be looked up.

- One Dynatrace problem can create duplicate PagerDuty incidents on the same service.
- Example: `#12924` and `#12925` both came from Dynatrace problem `P-260311737` on `customer-portal-projections`.
- Implication: investigators should search for concurrent incidents tied to the same external problem id before assuming separate outages.

- Environment-scoped problems can mix ownership and symptoms from different services.
- Example: `#12930` / `P-260311750` combined `Customer-Portal-Projections Projection Lag Above 1000` with `production DE Approval Rate <= 45%`.
- Implication: keep the routed PagerDuty service as the document owner by default, but explicitly document mixed ownership instead of forcing a clean single-service outage narrative.

- Low-load alerts are not outage proof.
- Example: `#12927` on `referrals` opened on low load without onset-matching service errors. The stronger worker errors appeared much later, so they were better treated as secondary symptoms.
- Implication: check for genuine low customer interaction, producer slowdown, backlog growth, or consumer fault before writing a low-traffic incident as a service outage.

- Chronic defects must be separated from acute onset.
- Example: `customer-portal-projections-recovery-primary` had been crash-looping since at least `2026-03-25` on `Invalid object name 'CustomerPortalRecovery.CheckPoints'`, well before the later lag incidents.
- Implication: chronic failures may matter for customer impact and follow-up work, but they should not be promoted to onset cause unless their timing changes in the incident window.

- Custom metric names can hide misleading monitor semantics.
- Example: `qp.decision_engine.checkout.risk_assessment` sounded like a simple approval-rate metric, but code inspection showed it is emitted as a `Distribution` of `Order.Amount`.
- Implication: the `<= 45%` approval alert was consistent with amount-weighted approval dips even while count-based approvals stayed around `50%` and service availability remained healthy.

- The routed service is not always the dominant breached metric owner.
- Example: the environment-level `zip.projections.es.lag` signal was often dominated by `platform-test-harness-projections`, not `customer-portal-projections`.
- Implication: investigators should compare the routed service slice against the top breached series before concluding that the routed service is the true root-cause owner.

## Practical Result

- The workflows now classify alert shape earlier.
- The Dynatrace guidance now distinguishes low-load noise, chronic background defects, environment-scoped ownership mismatch, and custom-metric semantics.
- The incident page template now captures alert scope, duplicate incidents, metric semantics, and ownership assessment explicitly.
