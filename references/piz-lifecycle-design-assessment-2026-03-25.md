# Assessment: PayInZ Lifecycle Automation Design

- Reviewed artifact: Confluence page `Tech Design & Scoping: Automation of Pi8 lifecycle (PayInZ Scaling)`, page id `5261688839`, version `6`, updated `2026-03-25`
- Assessment date: `2026-03-25`
- Reviewer input incorporated: supplied findings summary
- Recommendation: `Do not approve as written`

## Short Summary

- The design is pointed at the right problem, but Option A is not ready for approval.
- The biggest gaps are unclear transaction ownership, stale TTL-driven reevaluation, and an unresolved purchase-request bootstrap path.
- The persisted state model is too thin for multi-rail correctness, grandfathering, and safe retries.
- The safest revision is: SpendingPower owns the authoritative reevaluation transaction, Decision Engine stays assessment-only, and Purchase Request uses the same state-creation path.
- The doc should not move to implementation until it defines state identity, invalidation rules, bootstrap behavior, and the additive API contract.

## Executive Summary

The document is directionally correct about the business problem: PayInZ needs persisted lifecycle state, asymmetric re-eligibility logic, product-rail context, and additive API changes. The main issue is that the preferred solution, Option A, is not yet implementation-safe. It is internally inconsistent about transaction ownership, leaves the purchase-request bootstrap path unresolved, and relies too heavily on TTL-driven reevaluation for rules that are supposed to respond to customer behavior changes.

The safest path is to keep the service boundary split but tighten the transaction model:

1. SpendingPower should remain the single owner of eligibility state and the authoritative reevaluation transaction.
2. Decision Engine should remain a stateless assessment dependency that returns an assessment result only.
3. FEO should orchestrate reads, but not own the multi-step assess-and-persist transaction.
4. Purchase Request should use the same authoritative state-creation path as the exposure flow.
5. TTL should optimize reads, not define correctness. DPD, recency expiry, Nexus schedule changes, and grandfathering state transitions need explicit invalidation rules.

## Sign-Off Position

I would not sign off on Option A in its current form. The design needs to resolve three blockers before implementation starts:

1. Define a single authoritative owner for the read-assess-persist transaction.
2. Define the authoritative state identity and persisted schema.
3. Choose and fully specify the purchase-request bootstrap path.

Without those decisions, the system risks duplicate state creation, inconsistent eligibility decisions across flows, unclear retry semantics, and ambiguous downstream messaging.

## Findings

### 1. High: Preferred option is internally inconsistent about transaction ownership

The Option A endpoint table says FEO calls Decision Engine for assessment and SpendingPower for persistence. The detailed flow then says FEO calls SpendingPower `POST /anywhere/spending-limits`, and SpendingPower calls Decision Engine internally. Those are materially different designs.

This is not a wording issue. It changes:

- who owns retries and idempotency
- who sees and handles partial failure between assess and persist
- whether FEO or SpendingPower owns the reevaluation transaction
- whether NFR #10 service-boundary separation is actually being followed

The current wording leaves the preferred design with no single transaction owner.

### 2. High: TTL-based reevaluation is too weak for the rules the design claims to enforce

The design repeatedly relies on a `24hr` cache / reevaluation threshold. That is acceptable for amortizing repeated reads, but not for correctness when FR #2 requires eligibility removal on behavior changes such as `3DPD` or loss of recent-order recency.

With request-driven reevaluation only:

- a customer can remain incorrectly eligible until the next read
- transition messaging becomes delayed and non-deterministic
- analytics events reflect cache timing, not customer state timing
- stale state can survive until TTL expiry even when the business rule has already changed

The design needs explicit invalidation semantics for:

- `3DPD` threshold crossings
- recent-order recency expiry
- product schedule availability changes from Nexus
- grandfathering completion
- feature-flag or tier-config changes that intentionally alter assessment behavior

### 3. High: Purchase-request bootstrap path is unresolved

The `Purchase Request Flow` section acknowledges that some merchants never call SpendingPower `GET /anywhere/spending-power-limits`, so those customers may reach purchase-time logic with no persisted eligibility state. The design presents two options and does not choose one, even though FR #10 explicitly puts purchase-time behavior in scope.

That leaves several correctness questions unanswered:

- What creates authoritative state for first-touch customers?
- How is duplicate first-write prevented when exposure flow and purchase flow race?
- How does Purchase Request stay behaviorally identical to the FEO flow?
- What happens when there is no prior state and ML is unavailable?

Until the bootstrap path is chosen and merged into the main design, the system has no safe authoritative initialization flow.

### 4. High: Persisted data model is underspecified for the required lifecycle

The shared data model is currently:

```text
CustomerId
CurrentStatus
InstallmentCount
InstallmentIntervalDays
MerchantId
CreatedAt
UpdatedAt
```

That model is too thin for FR #4, FR #6, FR #8, and FR #13. It does not clearly encode:

- authorized party / rail identity
- stable schedule or product-offer identity
- Nexus config version or effective schedule version
- last assessment type and version
- last assessment timestamp and next reevaluation timestamp
- transition reason and transition timestamp
- grandfathered-once state
- request id / idempotency key / row version for concurrency control
- source flow used to initialize state

Without those fields, multi-rail correctness, grandfathering, auditability, and safe retries are all underspecified.

### 5. Medium: Grandfathering is stated as a requirement but not operationalized

FR #8 requires a one-time legacy assessment path for existing customers meeting specific DOB and tier rules, followed by permanent use of the new assessment. The design does not show where that one-time transition is recorded or how concurrent first-touch requests avoid repeating it.

This needs persisted, idempotent state, not just flow prose. At minimum the design needs:

- a persisted marker that legacy bootstrap has completed
- exact criteria for when the one-time path is allowed
- dedupe behavior for retries and concurrent first requests
- clear behavior when the first request fails after assessment but before persist

### 6. Medium: API contract is too vague for additive compatibility

Option A says the response is `Dictionary<MerchantId, List<ReEvaluationSchedule>>`, but the example shows a single object-like schedule payload rather than a list. The doc also does not precisely define:

- what old consumers see when no eligibility state exists
- whether `payInZEligibilityStatus` can be null, omitted, or `Unknown`
- whether `isEligibilityStatusChanged` means changed since last persisted state, last request, or last client-visible response
- whether `shouldReevaluateEligibility` is internal-only or part of the public additive contract
- how multiple schedules per merchant are represented

This is likely to create implementation drift across teams before coding starts.

### 7. Medium: End-to-end SLO and fallback semantics are not credible yet

The doc reuses service-local latency targets but does not define an end-to-end miss-path budget for the preferred flow. On a reevaluation miss, the current design implies:

- SpendingPower `GET`
- Decision Engine assessment, with possible ML call
- SpendingPower persist
- FEO response assembly

Using the document's own numbers, the miss path can easily become a multi-second user-path call. That may be acceptable, but it is not explicitly budgeted or defended.

The fallback story is also incomplete. NFR #4 says to return last persisted state when ML is unavailable, but behavior differs materially for:

- customers with no existing state
- currently eligible customers losing eligibility based only on rules
- currently ineligible customers requiring ML for re-eligibility

The design needs an explicit fallback matrix, not one sentence.

## Recommended Design Corrections

### 1. Use a single authoritative reevaluation transaction in SpendingPower

Keep the read path pure and additive:

- `GET /anywhere/spending-power-limits` reads state and returns current eligibility plus reevaluation metadata

Define one authoritative write-style reevaluation endpoint in SpendingPower:

- `POST /anywhere/spending-power-eligibility/re-evaluate`

That endpoint should:

1. load current state
2. decide whether reevaluation is required
3. call Decision Engine for assessment when needed
4. persist the new state atomically
5. emit transition events
6. return the final client-visible state

Decision Engine should not persist. FEO should not own the assess-persist transaction. Purchase Request should call this same state-management path when bootstrap is required.

### 2. Separate correctness invalidation from read optimization

Keep TTL for read amortization, but define invalidation triggers that force earlier reevaluation or state changes:

- `3DPD` transition events
- recent-order recency expiry events or scheduled sweeps
- Nexus schedule availability changes
- grandfathering completion
- explicit config version changes that materially affect assessment behavior

If exact real-time invalidation is not feasible for every rule, the design should say which transitions are event-driven and which remain TTL-driven, with an explicit risk tradeoff.

### 3. Strengthen the persisted model

The persisted record should be keyed by an explicit eligibility identity, not an implicit merchant-only key. At minimum the schema should include:

- `CustomerId`
- `MerchantId`
- `AuthorizedPartyId` or equivalent rail identity
- `ScheduleKey` or versioned product-offer key
- `CurrentStatus`
- `LastAssessmentMode` (`legacy` or `new`)
- `LastAssessmentVersion`
- `LastAssessedAt`
- `NextReevaluationAt`
- `TransitionReason`
- `TransitionedAt`
- `GrandfatheringCompletedAt`
- `RowVersion` or equivalent optimistic concurrency field
- `LastMutationRequestId` or equivalent idempotency token

### 4. Choose one bootstrap path and make it authoritative

The design should explicitly pick one of these:

- Purchase Request calls SpendingPower reevaluation endpoint when no state exists.
- A separate precompute job guarantees state existence before Purchase Request.

The first option is lower coordination and probably more realistic for initial delivery, but only if it uses the same authoritative persistence transaction as the FEO flow.

### 5. Tighten the API contract before implementation

Before build work starts, the design should define:

- exact JSON shape, including collections
- nullability and default behavior for legacy consumers
- whether `Unknown` is a valid status
- exact semantics for `isEligibilityStatusChanged`
- exact semantics for `shouldReevaluateEligibility`
- behavior when no state exists and initialization fails

## Bottom Line

The design has the right business intent, but it is still one design iteration away from being implementation-safe. The main issue is not whether persisted lifecycle state should exist; that part is clear. The issue is that the current preferred option does not yet define one authoritative transaction model across exposure reads, purchase requests, reevaluation, and grandfathering.

If the document is revised to:

1. make SpendingPower the authoritative reevaluation transaction owner
2. keep Decision Engine stateless for assessment only
3. define event-driven invalidation for non-TTL-safe rules
4. choose a single bootstrap path for first-touch customers
5. strengthen the state model and response contract

then the design would be much closer to approval.
