# Assessment: PayInZ Lifecycle Automation Design

- Reviewed artifact: Confluence page `Tech Design & Scoping: Automation of Pi8 lifecycle (PayInZ Scaling)`, page id `5261688839`, version `14`, updated `2026-03-25`
- Assessment date: `2026-03-26`
- Prior assessment status: this replaces the earlier review of version `6`
- Recommendation: `Do not approve as written`

## Short Summary

- The selected `Option A+B Hybrid` is the right architectural direction.
- The core flow is now much better aligned: FEO owns the read trigger, SpendingPower owns the authoritative reevaluation transaction, and Decision Engine remains assessment-only.
- The exposure miss path and Purchase Request bootstrap path now converge on the same SpendingPower `POST /anywhere/spending-power-eligibility/re-evaluate` endpoint, which is the correct shape.
- The remaining issues are no longer about picking the right top-level design. They are about making the selected hybrid implementation-safe.
- The biggest gaps are reevaluation semantics, the `DE -> SpendingPower -> DE` Purchase Request call chain, the persisted state model, the additive response contract, and the miss-path fallback/SLO story.

## Executive Summary

This revision is materially better than the earlier document. The selected hybrid now matches the safest design boundary:

1. FEO handles the normal read path.
2. SpendingPower owns the single authoritative reevaluation transaction.
3. Decision Engine performs assessment only and does not persist lifecycle state.
4. Purchase Request uses the same SpendingPower reevaluation endpoint instead of inventing a second bootstrap flow.

That resolves the biggest earlier concern around transaction ownership and significantly improves the overall design. The document is now close to approval, but it still leaves several implementation-critical details underspecified.

The main remaining issue is not "which option should we pick." The main issue is whether the chosen hybrid is specified tightly enough that multiple teams can build it consistently and safely. Today it still is not.

## Sign-Off Position

I would not sign off on the design as written yet, even though the selected hybrid is the correct direction.

Before implementation starts, the doc should still resolve these blockers:

1. define reevaluation based on persisted freshness, not just cache presence
2. define the `DE -> SpendingPower -> DE` Purchase Request path so it cannot recurse or produce inconsistent retries
3. define the authoritative persisted eligibility schema, including concurrency and grandfathering fields
4. define the exact additive response contract and missing-state semantics
5. define the end-to-end miss-path latency and fallback matrix

Without those details, the implementation still risks duplicate reevaluations, recursive dependencies, unnecessary DE/ML calls, ambiguous client behavior, and inconsistent state initialization across flows.

## Findings

### 1. High: The selected hybrid is correct, but reevaluation semantics still conflate cache miss with reevaluation required

The chosen flow says:

- FEO calls SpendingPower `GET /anywhere/spending-power-limits`
- on cache miss / stale or missing state, FEO calls SpendingPower `POST /anywhere/spending-power-eligibility/re-evaluate`

That is directionally correct, but the wording still reads as if `cache miss` and `re-evaluation needed` are the same thing. They are not.

If Redis is cold but the database contains a still-fresh persisted state, SpendingPower should be able to return that state and rebuild cache without forcing reassessment. Otherwise reevaluation frequency becomes dependent on cache eviction patterns instead of business freshness rules.

The design needs persisted freshness semantics such as:

- `LastAssessedAt`
- `NextReevaluationAt`
- optional explicit `stateMissing` signal

It should also define the behavior for:

- cache miss + fresh DB state
- cache miss + stale DB state
- no DB state
- concurrent reevaluation already in progress

Without that, the selected flow will cause avoidable DE and ML traffic and will be harder to reason about operationally.

### 2. High: Purchase Request now uses the correct shared SpendingPower bootstrap path, but the `DE -> SpendingPower -> DE` loop is still underspecified

Using the same SpendingPower reevaluation endpoint for Purchase Request is the right design decision. It avoids the earlier problem of having one authoritative bootstrap path for exposures and a different one for purchase requests.

However, the current flow still creates a synchronous call chain:

- Purchase Request logic in DE needs authoritative PiZ state
- DE calls SpendingPower
- SpendingPower calls DE for assessment

That can work, but only if the document explicitly states that SpendingPower calls a pure assessment endpoint that does not re-enter Purchase Request model-building logic or any higher-level orchestration path.

The doc still needs to define:

- that the DE endpoint invoked by SpendingPower is assessment-only and non-recursive
- who owns retries and circuit-breaking on this chain
- timeout budgets across the DE -> SpendingPower -> DE roundtrip
- idempotency/concurrency behavior when exposure flow and PR flow race to initialize the same customer state
- fallback behavior when there is no prior persisted state and DE or ML is unavailable

Until that is specified, the hybrid still has a material integration risk in the Purchase Request path.

### 3. High: The persisted eligibility data model is still too thin for the lifecycle the selected hybrid depends on

The current model is still effectively:

```text
Id
CustomerId
CurrentStatus
InstallmentCount
InstallmentIntervalDays
MerchantId
CreatedAt
UpdatedAt
```

That is not sufficient for the now-selected design, because SpendingPower is expected to be the authoritative owner of:

- reevaluation timing
- multi-rail identity
- grandfathering state
- transition tracking
- retry safety
- auditability across exposure and purchase flows

At minimum the design should define fields for:

- `CustomerId`
- `MerchantId`
- `AuthorizedPartyId` or equivalent rail identity
- `ScheduleKey` or versioned product-offer identity
- `CurrentStatus`
- `ReasonCode`
- `TransitionedAt`
- `TransitionReason`
- `LastAssessmentMode` (`legacy` or `new`)
- `LastAssessmentVersion`
- `LastAssessedAt`
- `NextReevaluationAt`
- `GrandfatheringCompletedAt` or equivalent one-time legacy bootstrap marker
- `RowVersion` or another optimistic concurrency token
- `LastMutationRequestId` or equivalent idempotency key
- optional `InitializedByFlow` for audit/debugging

The schema also needs to say what the record identity actually is, for example whether it is one row per `customer + merchant + rail + schedule`.

Without that, the selected hybrid still lacks a safe authoritative state definition.

### 4. Medium: The additive response contract is still ambiguous

The doc says the response is:

```text
Dictionary<MerchantId, List<ReEvaluationSchedule>>
```

but the example payload shows a single object-like value per merchant, not a list.

That mismatch is not editorial. It affects how FEO, Purchase Request, and downstream consumers will implement parsing and mapping.

The contract still needs to define:

- the exact JSON shape with an example that matches the declared type
- whether multiple schedules per merchant are possible and how they are represented
- whether missing state is represented as omitted, `null`, `Unknown`, or a dedicated `stateMissing` flag
- whether `shouldReevaluateEligibility` is an internal orchestration hint, part of the public additive contract, or both
- whether `isEligibilityStatusChanged` means changed relative to prior persisted state, changed during the current reevaluation transaction, or changed since last client-visible response
- what legacy consumers see when no state exists or reevaluation fails

This is still a real cross-team implementation risk.

### 5. Medium: The miss-path SLO and fallback semantics are still underspecified for the selected hybrid

The doc now has the right cache-hit and reevaluation shapes, but it still does not budget the end-to-end miss path explicitly.

For the selected hybrid, the user-visible miss path can involve:

- FEO -> SpendingPower `GET`
- SpendingPower DB load
- FEO -> SpendingPower `POST /re-evaluate`
- SpendingPower -> DE assessment
- optional DE -> ML call
- SpendingPower persist/cache/event emission
- SpendingPower -> FEO final response

That is a multi-step synchronous path. It may still be acceptable, but the document should defend it explicitly.

It also needs a clearer fallback matrix for at least these cases:

- no prior persisted state + ML unavailable
- existing eligible state + rules-only reevaluation says customer should become ineligible
- existing ineligible state + reevaluation needs ML for re-eligibility
- DE timeout after SpendingPower has loaded state but before persist

The doc should state whether SpendingPower returns:

- last persisted state
- an explicit "state unavailable / reevaluation failed" result
- or an error

It should also clarify that transition events are emitted only for successfully persisted transitions, not failed assessment attempts.

### 6. Low: The page should clean up legacy option labeling so the selected hybrid reads as the single decision record

The team has already selected the hybrid. Because of that, the remaining `Option A` section should no longer read as `Preferred`.

That is not the main technical issue anymore, but it is still worth fixing because it makes the page look partially undecided even though the selected-solution section already makes the architectural choice.

The simplest cleanup is to relabel the older options as:

- alternatives considered
- superseded options

That will make the doc easier for later readers to interpret.

## Recommended Design Corrections

### 1. Make persisted freshness the source of truth for reevaluation

Keep the chosen hybrid, but define reevaluation based on persisted freshness metadata rather than cache presence.

The doc should define:

- `LastAssessedAt`
- `NextReevaluationAt`
- whether `stateMissing` is part of the GET response
- whether SpendingPower can rebuild cache from a fresh DB record without reassessment
- whether the POST re-checks freshness/idempotency before calling DE

### 2. Tighten the Purchase Request reevaluation contract

Keep Purchase Request on the same SpendingPower `POST /anywhere/spending-power-eligibility/re-evaluate` endpoint.

The doc should explicitly state:

- SpendingPower calls a pure DE assessment endpoint
- that endpoint does not re-enter PR model building
- who owns retries, timeout budgets, and circuit-breaking
- how concurrent first-touch requests are deduped
- what happens when there is no prior state and DE or ML is unavailable

### 3. Strengthen the persisted schema

The design should define the authoritative eligibility identity and lifecycle fields up front.

At minimum it should include:

- rail identity
- schedule/product identity
- last assessment metadata
- reevaluation timing
- transition metadata
- grandfathering completion marker
- optimistic concurrency or idempotency fields

### 4. Tighten the additive API contract

Before implementation starts, the doc should define:

- one exact JSON example that matches the declared type
- nullability and missing-state semantics
- exact semantics of `isEligibilityStatusChanged`
- exact semantics of `shouldReevaluateEligibility`
- legacy-consumer behavior on no-state and reevaluation-failure scenarios

### 5. Add an explicit miss-path latency and fallback matrix

The doc should define:

- cache-hit latency expectations
- DB-hit-without-reevaluation expectations
- DE-only reevaluation expectations
- DE + ML reevaluation expectations
- fallback behavior for each failure class

## Suggested Inline Comments For The Design Doc

### `Selected Solution: Option A+B Hybrid — FEO-Triggered, SpendingPower-Orchestrated`

The hybrid looks like the right architecture and is much stronger than the earlier version because it now gives us one authoritative state-management path: FEO decides when to trigger, SpendingPower owns the reevaluation transaction, and DE stays assessment-only. I’d like the doc to be more explicit that this is the chosen design, not just the current leading option. For implementation, this section should act as the decision record: FEO owns the read trigger only, SpendingPower owns load/check-assess-persist-cache-events, and Purchase Request uses the same SpendingPower reevaluation path rather than a second bootstrap flow.

### `Flow 2: Stale or missing state (cache miss — re-evaluation needed)`

This flow is close, but the doc currently conflates `cache miss` with `re-evaluation required`. Those are not the same thing. A cold cache should not force reassessment if SpendingPower can read a still-fresh persisted state from DB. The design should define reevaluation based on persisted freshness metadata such as `LastAssessedAt` / `NextReevaluationAt`, and it should clarify behavior for cache miss + fresh DB state, cache miss + stale DB state, no DB state, and concurrent reevaluation already in progress.

### `Flow 3: Purchase Request path`

Using the same SpendingPower `POST /.../re-evaluate` endpoint for Purchase Request is the right direction because it avoids a second bootstrap path. The part that still needs tightening is the `DE -> SpendingPower -> DE` call chain. The doc should explicitly state that SpendingPower calls a pure DE assessment endpoint that does not re-enter Purchase Request model building, and it should define timeout, retry, concurrency, and fallback behavior for first-touch customers.

### `Eligibility Data Model`

The selected flow is now centered on SpendingPower as the authoritative transaction owner, but the persisted model is still too thin to support that safely. The current fields are enough to store a simple status, but not enough to support multi-rail identity, grandfathering, reevaluation decisions, auditability, or safe retries. This section should define rail identity, schedule/product identity, last assessment metadata, next reevaluation time, grandfathering marker, and concurrency/idempotency fields.

### `Response shape`

The contract here still needs to be tightened before implementation. The text says `Dictionary<MerchantId, List<ReEvaluationSchedule>>`, but the example shows a single object per merchant rather than a list. The doc should define one exact payload shape, clarify whether missing state is `null`, omitted, `Unknown`, or a dedicated flag, and define the exact semantics of `isEligibilityStatusChanged` and `shouldReevaluateEligibility`.

### `Non-Functional Requirements`

The availability and latency direction makes sense, but the doc should be more explicit about the end-to-end miss path now that the hybrid has been selected. The cache-hit path is easy; the key question is what happens on stale or missing state when SpendingPower has to call DE and possibly ML. This should be described as a user-path budget with a clear fallback matrix, especially for `no prior state + ML unavailable`.

### `Option A: FEO-Orchestrated, Split Assessment & State (Preferred)`

Since the team has already selected the hybrid, this section should not still read as the preferred path. I’d mark it as an alternative considered or a superseded option so the page reads as one chosen design rather than several partially-active candidates.

## Bottom Line

The updated design is much closer to approval than the prior version because it now converges on the correct top-level shape:

1. normal exposure reads use SpendingPower `GET`
2. stale or missing state uses FEO `GET` + SpendingPower `POST /re-evaluate`
3. Purchase Request uses the same SpendingPower reevaluation path

That is the right direction.

I still would not approve the document as written yet, because the remaining issues are implementation-critical rather than cosmetic. If the doc tightens reevaluation semantics, the Purchase Request call chain, the persisted schema, the response contract, and the miss-path failure model, then it should be in good shape for sign-off.
