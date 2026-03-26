# Assessment: DQ-8931 Extended Fraud Alert Handling with KRM

- Reviewed artifact: Confluence page `DQ-8931 Extended Fraud Alert Handling with KRM`, page id `5233115228`, version `2`, updated `2026-03-26`
- Assessment date: `2026-03-26`
- Prior assessment status: none
- Recommendation: `Do not approve as written`

## Short Summary

- The document is reasoning about the problem as if EFA ownership were mostly greenfield, but the codebase already has EFA persistence, underwriting semantics, and internal update endpoints.
- The main open issue is not whether to create "an EFA service" in the abstract. The real issue is where the cross-bureau workflow and Kustomer correlation state should live.
- There is already Kustomer integration in Front End Orchestration, but it is limited and customer-authenticated. The page should compare required Kustomer operations against that existing surface rather than discuss a generic gateway hypothetically.
- The proposed callback path is currently underspecified. It does not say how Kustomer will authenticate, how updates are correlated to the correct bureau-owned EFA record, or how retries and out-of-order callbacks are handled.
- Current EFA rule behavior in Decision Engine is already established. The design should explicitly preserve or intentionally change those semantics instead of assuming they will carry over.

## Executive Summary

This page is directionally aiming at the right operational goal: remove the manual CX and Decisioning workflow and automate the handoff through Kustomer. The problem is that the document starts from an incomplete picture of the current system.

Today, EFA is already represented in production code in three places:

1. underwriting rules in Decision Engine
2. bureau-specific persisted status in provider gateways
3. a limited Kustomer integration surface in Front End Orchestration

That matters because the design alternatives in the page are framed as if the team is deciding from scratch where tables, logic, and ingress should live. The codebase already answers part of that question:

- TransUnion has a dedicated `TransUnion.ExtendedFraudAlerts` table and internal update API.
- Experian persists EFA state on `CreditBureau.ComplianceFeatures` and exposes an internal update API.
- Decision Engine already depends on those statuses when deciding whether a customer may proceed.
- Front End Orchestration already contains Kustomer API code, but not the full set of capabilities this workflow appears to need.

Because of that, I would not approve the page as written yet. The next revision needs to stop treating EFA automation as a blank-slate service placement exercise and instead define one explicit ownership model:

1. what remains the authoritative underwriting state
2. what new workflow/correlation state is required for Kustomer automation
3. which service owns inbound callback handling and outbound Kustomer orchestration
4. how that owner updates the existing bureau-specific records safely

## Sign-Off Position

I would not sign off on the design as written.

Before implementation starts, the doc should still resolve these blockers:

1. correct the current-state inventory so it reflects the real EFA persistence and service boundaries in code
2. define the authoritative state model and clearly separate bureau underwriting state from Kustomer workflow state
3. specify the exact Kustomer capability boundary, including whether existing FEO integration is extended or a new narrowly scoped service is introduced
4. define the callback/ingress contract, correlation model, state machine, and idempotency behavior
5. define reliability, retry, reconciliation, and migration/backfill behavior

Without that, the implementation risks duplicating existing EFA storage, introducing split-brain state across services, and creating a fragile third-party callback path that cannot safely update the actual underwriting status used by Decision Engine.

## Findings

### 1. High: The current-state section is materially incomplete; EFA persistence and status ownership already exist

The document discusses whether an EFA service should "own" the tables that store EFA status and suggests those tables currently live with Decision Engine. That is not an accurate description of the current system shape.

Concrete code evidence:

- TransUnion already has a dedicated `TransUnion.ExtendedFraudAlerts` table created by migration in `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Data/Migrations/20240122153140_Initial.cs:16-48`, later evolved in `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Data/Migrations/20240313161246_AddKeyToRowNumberForTransUnionFraudAlertRowNumber.cs:23-35` and `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Data/Migrations/20250131154712_ExtendedIdentityFraudCode.cs:15-21`.
- That table is actively mapped in `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Data/TransUnionDbContext.cs:24-73` and backed by the DTO in `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Data/Dtos/ExtendedFraudAlertDto.cs:7-24`.
- Experian persists EFA state on `CreditBureau.ComplianceFeatures` via `provider-gateways/src/QuadPay.ProviderGateways.Experian/Data/ExperianGatewayDbContext.cs:21-80` and `provider-gateways/src/QuadPay.ProviderGateways.Experian/Data/ComplianceFeatureSet.cs:5-20`, with expiration added by `provider-gateways/src/QuadPay.ProviderGateways.Experian/Data/Migrations/20230425192721_AddEfaExpirationDate.cs:18-24`.
- Decision Engine already orchestrates status updates through `decision-engine/src/QuadPay.DecisionEngine/CreditBureaus/Services/ExtendedFraudAlertStatusManager.cs:29-85`.

This matters because the design is currently evaluating service placement without first acknowledging that bureau-owned persisted status already exists and is already coupled to underwriting decisions. The page needs to update its "existing state" section so later architectural choices are based on the real baseline.

### 2. High: The document does not define the authoritative boundary between bureau underwriting state and new Kustomer workflow state

The existing persisted records are bureau-specific and relatively thin:

- TransUnion stores `CustomerId`, `Message`, `Status`, `ExtendedIdentityFraudCode`, `ExpiresOn`, and audit timestamps.
- Experian stores EFA-related fields on `ComplianceFeatures`.

Those records are sufficient for underwriting, but they are not obviously sufficient for the proposed automated workflow, which introduces new concerns:

- Kustomer conversation/note/workflow identifiers
- callback correlation
- last outbound create/update attempt
- callback delivery state
- actor/source of manual or workflow-driven updates
- allowed workflow transitions
- replay/idempotency tokens

The page currently talks about "updating the customer's status in the database immediately" without saying which database record is authoritative for which concern.

That needs to be split explicitly:

1. authoritative underwriting state used by rules
2. workflow/correlation state needed to operate the Kustomer process

If the design wants to keep bureau-owned EFA status as the source of truth for underwriting, that is fine, but then any new persistence should be explicitly described as workflow metadata rather than a replacement EFA table. If the design wants a new canonical EFA workflow record, it needs to explain how that record maps to and updates the existing bureau-specific stores.

Right now it does neither, which creates split-brain risk.

### 3. High: The Kustomer gateway discussion is not grounded in the capabilities that already exist in Front End Orchestration

The page evaluates a possible dedicated Kustomer CRM gateway as though Kustomer integration were absent today. That is not the case.

Concrete code evidence:

- FEO already exposes Kustomer endpoints in `frontend-orchestration/src/QuadPay.FrontEndOrchestration/Controllers/v3/KustomerController.cs:17-150`.
- Its current Kustomer API surface is limited to `GetConversation` and `AppendTagsToConversation` in `frontend-orchestration/src/QuadPay.FrontEndOrchestration.Application/Services/IKustomerApi.cs:13-25`.
- Its service layer supports `SendDraftAsync`, `GetCustomerByExternalIdAsync`, and `UpdateConversationAsync` in `frontend-orchestration/src/QuadPay.FrontEndOrchestration.Application/Services/IKustomerService.cs:10-33` and `frontend-orchestration/src/QuadPay.FrontEndOrchestration.Application/Services/KustomerService.cs:41-215`.

What I did not find in the codebase:

- a Kustomer conversation-creation API
- a Kustomer note-creation API matching the page's external references
- a Kustomer webhook/workflow callback controller
- an internal system-to-system Kustomer orchestration surface

That means the design should not discuss "build a gateway or not" generically. It should list the exact operations DQ-8931 needs, compare them against the existing FEO integration, and then make a scoped decision:

- extend FEO with the missing internal capabilities
- or build a narrowly scoped EFA/Kustomer orchestration component

What it should not do is justify a new org-wide gateway based on possible future reuse. The current page does not yet make that distinction.

### 4. High: The ingress/callback design is underspecified, and this is the highest implementation-risk area in the page

The document says a callback endpoint will update the database after the CX agent finishes the call, but it does not define the hard parts:

- how Kustomer or Kustomer Workflows authenticates to the endpoint
- how retries are handled
- how duplicate callbacks are deduplicated
- whether out-of-order updates are possible
- how the callback identifies the correct customer and the correct bureau-owned EFA record
- what the allowed state transitions are
- whether the update is synchronous or queued

This is not an editorial gap. It is the integration design.

Concrete code evidence:

- TransUnion already exposes an internal update endpoint at `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Controllers/ExtendedFraudAlertController.cs:12-39`.
- Experian already exposes an internal update endpoint at `decision-engine/src/QuadPay.DecisionEngine/Services/CreditBureau/HttpGateway/IHttpCreditBureauGateway.cs:33-34`, implemented by `provider-gateways/src/QuadPay.ProviderGateways.Experian/CreditBureauReports/CreditBureauReportsController.cs:67-79`.
- Decision Engine rule models depend on `SelectedCreditBureau` and read EFA status through the aggregate in `decision-engine/src/QuadPay.DecisionEngine/CreditBureaus/Aggregation/Models/CreditReportAggregate.cs:19-96`.

Because the persisted status is split by bureau, the page must define how the callback owner determines where the authoritative status update belongs. Today that answer is absent.

### 5. Medium: The design does not explicitly preserve the current underwriting semantics that depend on EFA status

The page is focused on operational automation, but the current EFA statuses already have direct underwriting meaning.

Concrete code evidence:

- Checkout treats `CustomerApproved` as pass, `CustomerDeclined` as fail, and `CustomerAwaiting` / `CustomerNotAvailable` as fail while the alert is unexpired in `decision-engine/src/QuadPay.DecisionEngine/Services/RiskAssessments/Products/Checkout/Rules/CustomerWithExtendedFraudAlertMustHaveBeenContactedRule.cs:31-42`.
- QPAW purchase request follows the same shape in `decision-engine/src/QuadPay.DecisionEngine/Services/RiskAssessments/Products/QPaw/PurchaseRequest/Rules/CustomerWithExtendedFraudAlertMustHaveBeenContactedRule.cs:29-40`.
- Decision Engine also rewrites some future-dated `CustomerNotAvailable` cases back to `CustomerAwaiting` in `decision-engine/src/QuadPay.DecisionEngine/CreditBureaus/Services/ExtendedFraudAlertStatusManager.cs:39-50`.

The new design should say explicitly whether these semantics remain unchanged. If the callback workflow changes timing or meaning around `CustomerNotAvailable`, for example, that changes underwriting behavior, not just CX process automation.

The page currently treats status update as operational bookkeeping when it is actually decisioning-critical state.

### 6. Medium: The design needs an explicit reliability model because Kustomer is a third-party dependency, and the current code is not sufficient for guaranteed automation

The proposed workflow replaces manual work with third-party integration. That means reliability needs to be designed, not implied.

Concrete code evidence:

- `frontend-orchestration/src/QuadPay.FrontEndOrchestration.Application/Services/KustomerService.cs:50-90` and `:105-172` performs direct HTTP calls to Kustomer, logs warnings on failure, and returns `null`.

That is fine for the current limited usage pattern, but it is not enough for a workflow that is expected to create Kustomer artifacts automatically whenever a customer is marked declined due to EFA.

The page should state:

- whether Kustomer note/ticket creation is in-band or async
- retry policy
- dead-letter/replay strategy
- reconciliation/reporting process for failed outbound creates
- manual fallback for callback failures

Without that, the new system will still require manual operational recovery, just in a less visible way.

### 7. Medium: The document is missing migration, reconciliation, and observability requirements for in-flight EFA customers

The page explains why the current process is temporary and fragile, but it does not explain how the new system takes over from that process safely.

Missing items include:

- how existing customers already in `CustomerAwaiting`, `CustomerDeclined`, or `CustomerNotAvailable` are seeded into the new workflow
- how existing Kustomer/manual tickets are correlated, if at all
- whether historical active EFAs get notes created retroactively
- what metrics and alerts will exist for callback failures and unsynchronized states
- whether a unified audit/BI event is emitted when the automated workflow creates or resolves an EFA ticket

Today there is already some status-change eventing on the TransUnion side in `provider-gateways/src/QuadPay.ProviderGateways.TransUnionUS/Data/TransUnionModelRepository.cs:67-77` and `:113-163`, but the page does not yet turn that into an end-to-end operational story.

## Recommended Design Corrections

### 1. Fix the current-state inventory first

Before comparing alternatives, the page should document the actual current boundaries:

- Decision Engine owns rule evaluation and status-dependent underwriting behavior
- TransUnion owns a dedicated `ExtendedFraudAlerts` table and internal status update endpoint
- Experian owns EFA fields on `ComplianceFeatures` and an internal status update endpoint
- Front End Orchestration owns a limited Kustomer integration surface

That becomes the baseline for every later decision.

### 2. Define two explicit state domains

The design should separate:

- underwriting state: the status consumed by Decision Engine rules
- workflow state: Kustomer correlation and process metadata

If a new persistence model is introduced, it should be described as workflow/correlation state unless the team is intentionally replacing the current bureau-owned storage.

### 3. Make the Kustomer decision capability-driven, not hypothetical

The document should list the exact operations required for DQ-8931, for example:

- create or trigger a Kustomer conversation/workflow
- attach note or tags
- correlate outbound artifact back to customer and bureau
- receive callback/workflow completion

Then it should compare those requirements to the current FEO Kustomer surface and decide one of:

- extend FEO with missing internal capabilities
- or create a narrowly scoped orchestration surface specifically for EFA automation

I would avoid creating a generic org-wide Kustomer gateway as part of this initiative unless broader ownership and reuse are already committed.

### 4. Define the callback contract as a state machine

The page should specify:

- callback authentication
- idempotency key
- correlation identifier(s)
- allowed transitions among `CustomerAwaiting`, `CustomerApproved`, `CustomerDeclined`, and `CustomerNotAvailable`
- conflict handling for duplicate or late callbacks
- how the selected bureau is resolved
- who invokes the existing internal bureau update endpoints

This needs to be concrete enough that implementation can proceed without guesswork.

### 5. Keep Kustomer off the critical underwriting path

Creating a note or CRM artifact should not be required to complete the customer-facing decision path successfully. The page should explicitly use an async/outbox style approach with retries and reconciliation rather than making Kustomer availability part of the core assessment flow.

### 6. Add migration, observability, and reconciliation requirements

The doc should define:

- backfill strategy for active EFAs
- recovery playbook for failed outbound Kustomer actions
- recovery playbook for failed callbacks
- metrics, dashboards, and alerts
- audit/event requirements

## Suggested Inline Comments For The Design Doc

### `1 Existing State and Problem Statement`

This section should describe the current implementation more precisely. EFA is not just a manual process wrapped around a missing system. We already have bureau-owned persisted EFA status, Decision Engine rules that depend on that status, and limited Kustomer integration in FEO. The later service-placement discussion should build from that baseline.

### `3.1 EFA Service`

The pros/cons here are framed around whether the EFA business logic is "lightweight," but that is not the main architectural question. The real reason to introduce a dedicated component would be to own cross-bureau workflow correlation and Kustomer callback handling. If that is not the intent, then the section should say so. If it is the intent, the section should describe that explicitly rather than talking about owning a generic EFA process.

### `3.2 Kustomer CRM Gateway`

This section should compare exact required Kustomer operations against the existing FEO integration. Today we already have some Kustomer support in FEO, but not an obvious internal workflow surface, callback handler, or create-conversation/note contract. The decision should be based on that concrete gap analysis, not on speculative future reuse.

### `3.3 Ingress Point`

The ingress choice cannot just be preference. It determines who owns callback auth, idempotency, correlation, bureau resolution, and the state machine for updating the existing EFA records. This section should define those responsibilities explicitly.

### `2 Proposed Architecture`

The page should say that Kustomer artifact creation is asynchronous relative to the underwriting decision path, and it should describe what happens when Kustomer is unavailable or the callback never arrives.

## Bottom Line

The goal of the design is sound, but the document is still missing the implementation-defining details.

The biggest issue is that the page is evaluating service placement as if EFA status storage and Kustomer integration do not already exist. They do. The next revision needs to start from the real current-state boundaries and then define one clear workflow owner around them.

Until the doc does that, I would not approve it.
