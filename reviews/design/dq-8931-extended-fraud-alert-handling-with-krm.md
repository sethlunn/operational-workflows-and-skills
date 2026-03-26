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

## Suggested Review Comments For The Design Doc

### 1. `1 Existing State and Problem Statement`

**Location to comment on**

- Paragraph beginning `The existing state diagrammed above...`

**Comment to add**

This section should inventory the actual current implementation, not just the manual operational workflow. We already have bureau-owned persisted EFA status and Decision Engine behavior that depends on it, so the later service-placement discussion should start from that baseline.

**Clarification / likely follow-up explanation**

The current section reads as if EFA handling is mostly a manual process wrapped around a missing system. That is not quite true in the codebase today. We already persist EFA status in bureau-owned stores, and Decision Engine already consumes that status as underwriting input. That means the document is not starting from a greenfield architecture problem.

If the page does not call that out explicitly, the later alternatives can drift into the wrong framing, for example:

- proposing a new component to "own the EFA tables" when those tables or equivalent persisted fields already exist in bureau-owned systems
- describing callback ingress as a new problem without acknowledging that internal update paths already exist
- treating EFA as purely an operations workflow when it is already tied to live underwriting behavior

This comment is meant to push the page to define the real baseline before comparing options. Without that, the alternatives section risks optimizing around an inaccurate mental model of the current system.

### 2. `2 Proposed Architecture`

**Location to comment on**

- Sentence `Then, as soon as the CX agent finishes the call, the customer’s status will be updated in the database immediately.`

**Comment to add**

Which record is authoritative here? The doc should explicitly separate underwriting state from any new Kustomer workflow/correlation state; otherwise we risk split ownership between bureau EFA records and new automation metadata.

**Clarification / likely follow-up explanation**

This sentence sounds simple, but it skips the hardest modeling question in the design: what exactly is being updated, and which service owns that truth?

Today the underwriting-relevant EFA status already lives in bureau-specific persistence. The proposed workflow introduces additional state that does not obviously belong in those same records, such as:

- Kustomer conversation, workflow, or note identifiers
- callback correlation identifiers
- outbound delivery attempts and retry metadata
- source or actor of the latest update
- callback processing status and idempotency markers

Those concerns are different from the minimal status state used by underwriting. If the design does not explicitly separate them, the implementation can easily end up with two competing "sources of truth":

- one for the status Decision Engine reads
- one for the status the Kustomer workflow thinks it has written

This comment is asking the author to define that boundary up front. If bureau-owned records remain authoritative for underwriting, the doc should say so plainly and describe any new storage as workflow metadata only.

### 3. `2 Proposed Architecture`

**Location to comment on**

- Sentence `It also means that if future changes are made to the EFA process, they can live inside Kustomer itself.`

**Comment to add**

I’d be careful with this boundary. Kustomer can own workflow automation, but underwriting-critical semantics and authoritative EFA status should stay in our services. The doc should call that ownership split out explicitly.

**Clarification / likely follow-up explanation**

There is a difference between allowing Kustomer to automate operational steps and allowing Kustomer to become the effective owner of business state. The former is reasonable. The latter is risky.

The EFA statuses are not just CRM workflow labels. They have underwriting meaning in Decision Engine. Because of that, Kustomer should be treated as an integration participant, not as the canonical owner of whether a customer is in an approved, awaiting, declined, or not-available EFA state.

If the page says future EFA changes can "live inside Kustomer" without qualification, that invites ambiguity around:

- whether status transition rules move into Kustomer workflows
- whether the source of truth for a customer’s EFA outcome becomes a third-party system
- whether product or underwriting behavior can change indirectly through CRM config

This comment is not arguing against using Kustomer Workflows. It is asking the doc to make the boundary explicit: Kustomer may orchestrate tasks and emit callbacks, but our services should continue to own underwriting-critical semantics and persisted status.

### 4. `2 Proposed Architecture`

**Location to comment on**

- Sentence `This does, however, leave some questions as to how this should be implemented.`

**Comment to add**

We should add reliability, migration, and recovery requirements here: async/outbox behavior for outbound Kustomer work, retry/reconciliation for failed creates or missing callbacks, and a backfill plan for customers already in an active EFA state.

**Clarification / likely follow-up explanation**

Right now the page describes the happy-path workflow, but it does not describe what makes the automation production-safe. That gap is material because this design is replacing manual operational handling with a third-party integration.

At minimum, the page should answer:

- Is outbound Kustomer creation in-band with the customer decision flow, or async?
- What happens if Kustomer is down when we try to create the artifact?
- What happens if the artifact is created, but our system never records that fact?
- What happens if the CX-side callback is delayed, duplicated, or never sent?
- How do we take over customers already sitting in active EFA states when this launches?

Without those answers, the design is incomplete in a way that will surface later as operational toil. This comment is meant to force reliability and migration concerns into the design itself, instead of leaving them as implementation details.

### 5. `3.1 EFA Service`

**Location to comment on**

- Paragraph beginning `We could, instead, elect to create a separate EFA service.`

**Comment to add**

The main question here is less "is the logic lightweight?" and more "do we need one owner for cross-bureau workflow correlation, callback handling, and Kustomer orchestration?" If yes, say that explicitly. If no, this section should avoid implying a new service would replace the existing bureau-owned EFA persistence.

**Clarification / likely follow-up explanation**

The current pros/cons are framed around whether EFA logic is large enough to justify its own service. That is not the strongest architectural criterion here.

The real reason to introduce a dedicated owner would be if we want one component to be responsible for the new workflow-specific concerns that do not cleanly belong to either bureau integration or Decision Engine alone, for example:

- correlation across Kustomer and internal EFA records
- callback validation and idempotent processing
- centralized transition handling across TransUnion and Experian
- unified observability and reconciliation for the automation

If that is the actual motivation, the page should say so directly. If it is not, then the section should not imply that a new service would somehow become the natural owner of all EFA data, because the authoritative underwriting state already has bureau-specific ownership.

This comment is trying to sharpen the decision criterion so the design compares architectures on the real axis of value, not on an abstract "is the logic heavy enough?" question.

### 6. `3.2 Kustomer CRM Gateway`

**Location to comment on**

- First paragraph under `3.2 Kustomer CRM Gateway`

**Comment to add**

Before deciding on a new gateway, this section should compare the exact Kustomer operations needed for DQ-8931 against the Kustomer capabilities we already have. The decision should be based on that concrete gap analysis, not on speculative future reuse.

**Clarification / likely follow-up explanation**

The current text discusses a dedicated gateway as a general separation-of-concerns option, but it does not anchor that discussion in the actual Kustomer surface area required for this initiative.

That comparison matters because we are not starting from zero. We already have some Kustomer integration in FEO, but it appears limited. The design should enumerate the exact operations DQ-8931 needs, such as:

- create or trigger the Kustomer artifact/workflow
- attach note or tags if needed
- store or return a correlation identifier
- accept some form of callback or completion signal
- expose an internal system-to-system interface if customer-authenticated endpoints are not sufficient

Once those requirements are listed, the decision becomes concrete:

- extend the existing integration if the gap is small and ownership fits
- or build a narrowly scoped orchestration surface if the gap is substantial

This comment is pushing the author away from a hypothetical "org-wide Kustomer gateway" discussion and toward a capability-driven decision for this specific project.

### 7. `3.3 Ingress Point`

**Location to comment on**

- Sentence `This is largely preference...`

**Comment to add**

I don’t think this is just preference. The ingress owner determines callback auth, idempotency, duplicate/out-of-order handling, correlation to the correct customer and bureau-owned EFA record, and who invokes the existing internal update path. This section should define those responsibilities and the allowed status transitions.

**Clarification / likely follow-up explanation**

This is the highest-risk integration point in the design, because it is where a third-party-driven event becomes a write to underwriting-relevant internal state.

Calling it "preference" understates the design impact. The chosen ingress point becomes responsible for several implementation-defining behaviors:

- authenticating the caller
- validating payload shape and versioning
- resolving which customer and which bureau record should be updated
- deduplicating retries or repeated workflow callbacks
- rejecting invalid or late transitions
- deciding whether processing is synchronous or queued
- invoking the existing internal update path correctly

If the page does not define that contract, different implementations could make incompatible assumptions. This comment is asking for the ingress section to describe the state machine and ownership model, not just list candidate services.

### 8. `4 Questions/Notes from Team Review`

**Location to comment on**

- Placeholder text `(To be captured)`

**Comment to add**

Before approval, this section should capture the concrete unresolved decisions: authoritative state model, chosen owner for Kustomer orchestration, callback contract, reliability/reconciliation plan, and migration/backfill plan.

**Clarification / likely follow-up explanation**

This section is currently just a placeholder, but for a design in this state it should function as the explicit blocker list for approval. The page already surfaces open questions across multiple sections, but they are not consolidated into one actionable review checklist.

Capturing them here would make the review process much clearer by separating:

- informational notes
- implementation details that can wait
- design decisions that must be resolved before build starts

This also helps avoid the common problem where a design feels "directionally right" but still lacks the handful of concrete decisions that determine whether implementation will be safe and coherent.

## Bottom Line

The goal of the design is sound, but the document is still missing the implementation-defining details.

The biggest issue is that the page is evaluating service placement as if EFA status storage and Kustomer integration do not already exist. They do. The next revision needs to start from the real current-state boundaries and then define one clear workflow owner around them.

Until the doc does that, I would not approve it.
