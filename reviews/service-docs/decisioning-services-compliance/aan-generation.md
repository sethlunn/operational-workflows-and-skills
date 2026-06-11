# Compliance AAN Generation

- Service or system: `decisioning-services / compliance`
- Repo path: `services/compliance/src/Zip.Compliance/AAN`
- Exact scope: current-state adverse action notice generation from inbound bus message through MediatR orchestration, reason resolution, document selection, PDF storage, reference persistence, notification fanout, and retrieval APIs as reviewed on `2026-04-21`
- Audience: engineers debugging, extending, or refactoring adverse action notice generation in Compliance
- Last reviewed: `2026-04-21`
- Primary conclusion: Compliance does not make the underlying credit decision. It receives decline-oriented messages, normalizes them into MediatR commands, decides whether a notice should be skipped, delayed, deduplicated, or generated, resolves the customer-facing adverse-action language and document template, renders the PDF, stores it in blob storage, records a retrieval reference in Cosmos, and emits the downstream eligibility and sent events.

# Methodology

- Direct evidence from code:
  - `src/Zip.Compliance/Program.cs`
  - `src/Zip.Compliance/ServiceCollectionExtensions/AanServiceCollectionExtensions.cs`
  - `src/Zip.Compliance/ServiceCollectionExtensions/AuditLogDecoratorCollectionExtensions.cs`
  - `src/Zip.Compliance/AAN/Commands/CreateAdverseActionNotice.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/*.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/PendingAchUnderwritingAssessmentAanHandler.cs`
  - `src/Zip.Compliance/AAN/Services/*.cs`
  - `src/Zip.Compliance/AAN/Services/Resolvers/*.cs`
  - `src/Zip.Compliance/Data/Repositories/AanDocumentRepository.cs`
  - `src/Zip.Compliance/Data/Repositories/AanReferenceRepository.cs`
  - `src/Zip.Compliance/Data/Repositories/CustomerAchUnderwritingRepository.cs`
- Telemetry used:
  - none for this page; this page is intentionally code-backed and current-state only
- Interpretation:
  - where the code splits behavior by assessment type, feature flag, or bus message shape, this page describes the observed runtime branches rather than intended product policy

# Visual Flow Maps

- AAN generation overview:
  - source: `./diagrams/compliance-aan-generation-flow.mmd`
  - rendered snapshot: `./diagrams/rendered/compliance-aan-generation-flow.svg`
- Standard shared-handler sequence:
  - source: `./diagrams/compliance-aan-standard-sequence.mmd`
  - rendered snapshot: `./diagrams/rendered/compliance-aan-standard-sequence.svg`
- Delayed ACH AAN sequence:
  - source: `./diagrams/compliance-aan-ach-delay-sequence.mmd`
  - rendered snapshot: `./diagrams/rendered/compliance-aan-ach-delay-sequence.svg`

# What This Flow Owns

- It owns:
  - message ingestion inside Compliance after the bus route is already configured
  - normalization from bus event to MediatR command
  - AAN eligibility short-circuit behavior inside Compliance
  - duplicate suppression
  - adverse-action reason-string selection
  - template selection
  - PDF generation
  - blob storage writes
  - Cosmos reference persistence
  - downstream notification and BI fanout
  - customer-facing retrieval APIs
- It does not own:
  - the original lending, fraud, identity, or underwriting decision
  - the upstream rule execution that produced the decline
  - the upstream machine-learning models or credit-bureau providers
  - the caller experience in Decision Engine, checkout orchestration, or frontend systems

# Inbound Message Surface

Compliance wires Azure Service Bus in two layers.

- `Program.cs` registers queue `service-compliance` plus topic routes for `integration-events` and `compliance-events`.
- `AddAdverseActionNotices` adds the `AdverseActionNoticeEventHandlers` subscription on topic `compliance-events`.

The AAN generation surface currently includes these main inbound paths:

| Path | Bus message | First handler | Shared execution target | Notes |
| --- | --- | --- | --- | --- |
| App signup decline | `InitialExposureAssessmentDeclined` | `InitialExposureAssessmentDeclinedHandler` | `CreateAdverseActionNotice.Command` | App path can fetch ML reasons and can override mapped reasons from Socure reason codes. |
| Checkout purchase decline | `CheckoutPurchaseRequestDenied` | `CheckoutPurchaseRequestDeniedHandler` | `CreateAdverseActionNotice.Command` | Checkout path can fetch checkout ML reasons and carries `OrderBankPartner`. |
| Checkout prequal decline | `CheckoutPreQualificationDenied` | `CheckoutPreQualificationDeniedHandler` | `CreateAdverseActionNotice.Command` | Prequal fanout later publishes `CheckoutPreQualificationAanCreated`. |
| Mobile-app purchase decline | `MobileAppPurchaseRequestDeclined` | `MobileAppPurchaseRequestDeclinedHandler` | `CreateAdverseActionNotice.Command` | Skips entirely when `RulesRun` contains `ConfirmPurchase`. |
| Delayed ACH initial decline | `InitialExposureAssessmentDelayDeclined` | `AdverseActionNoticeHandler` | stored `CreateAdverseActionNotice.Command`, then later `CreateAdverseActionNotice.Command` again | This path schedules delayed delivery instead of generating immediately. |
| ACH underwriting final decline | `AchUnderwritingAssessmentDeclined` | `AchUnderwritingAssessmentDeclinedHandler` | `CashFlowUnderwritingUpdateAan.Command`, then `CreateAdverseActionNotice.Command` | Updates ACH state before creating the notice. |
| Direct ML decline | `MachineLearningAdverseActionDeclined` | `MachineLearningAdverseActionDeclinedHandler` | `CreateAdverseActionNotice.MachineLearningCommand` | This message is explicitly documented in code as a lightweight message received from Decision Engine. |
| Retroactive regeneration | `GenerateRetroactiveAan` | `GenerateRetroactiveAanHandler` | standalone retroactive path | This page focuses on the main live-generation paths, but retroactive generation reuses the same resolver and storage ideas. |

The important architectural shape is consistent with the mental model in the request: Compliance consumes bus messages, hands them to message-specific handlers, and those handlers convert the event payload into a MediatR command that runs the shared AAN generation logic.

# Shared Generation Flow

## 1. Normalize the inbound event into a command

Most ingress handlers do two things before calling MediatR:

- collapse the raw upstream failure state into one `HighestPriorityFailure`
- enrich the command with enough data for downstream document generation

That normalization happens in handler-specific code:

- app and mobile paths call `IEligibilityResolver.GetFailureRuleTypeApp(...)`
- checkout and prequal paths call `IEligibilityResolver.GetFailureRuleTypeCheckout(...)`
- ACH underwriting uses `GetFailureRuleTypeAchUnderwriting()`
- the direct ML path skips the eligibility resolver and trusts the inbound direct-ML event

The standard command shape is `CreateAdverseActionNotice.Command`. It carries:

- `CustomerId`
- `AssessmentId`
- `AssessmentType`
- `EventCreatedDate`
- `RulesRun`
- `DeclineReason`
- `FailureRuleType`
- credit data such as `CreditScore` and `CreditReports`
- optional `MachineLearningDeclineReasons`
- optional `SocureReasonCodes`
- optional `CustomerPersonalInfo`
- optional serialized `AssessmentModel`

The direct-ML path uses `CreateAdverseActionNotice.MachineLearningCommand` instead. That slimmer command keeps the already-resolved ML reasons, typed identity reason codes, bank-partner context, and credit data without replaying the standard eligibility-resolution step.

## 2. Resolve whether Compliance should generate anything at all

The shared standard handler starts by calling `AugmentCommand(command)`.

That helper:

- backfills `CustomerPersonalInfo`, `CreditScore`, and `CreditReports` from `Model`
- is especially important for delayed ACH messages that were serialized to the bus earlier
- creates a fresh `AanEligibilityResolution` whose `HighestPriorityFailure` is just `command.FailureRuleType`

From there, the shared handler branches early:

- if `FailureRuleType` is `RuleTypes.None`, it marks `IsNotHardDecline = true`, emits only the eligibility event, and returns
- if `FailureRuleType` is `RuleTypes.LockedOrFrozenCredit`, it logs that the customer will receive an AAN and continues through the normal generation flow

This means Compliance can receive a decline-shaped message and still intentionally not generate a PDF when the internal eligibility model says there is no customer-facing AAN to send.

## 3. Suppress duplicates before generating a new PDF

The standard path has two duplicate checks.

The generic duplicate check:

- loads the customer's `AdverseActionNoticeReference` from Cosmos through `IAanReferenceRepository`
- treats any notice inside a plus-or-minus 24 hour window around `EventCreatedDate` as already sent
- emits only the eligibility event with `PreviousAanWithin24Hours = true`
- skips document generation, storage, reference persistence, and sent notification

The ACH-specific duplicate check:

- calls `ICustomerAchUnderwritingRepository.HasAanSentWithinTimeframe(...)`
- uses the ACH underwriting record because ACH has additional status transitions that are not fully represented by the generic AAN reference list

The direct ML handler performs only the generic 24 hour dedupe check.

## 4. Turn failed-rule context into customer-facing adverse-action reasons

This is the step that converts decisioning-oriented rule identifiers into the strings that appear in the final document.

For the standard handler:

- `MappedRuleTypes.GetAdverseActionNoticeReasons(...)` walks the failed entries in `RulesRun`
- it keeps only rules whose mapped `RuleTypes` match the command's `FailureRuleType`
- it returns the distinct customer-facing adverse-action reason strings for those rules

There are then two important overrides:

- for `InitialExposureAssessmentDeclined` and `CheckoutPurchaseRequestDenied`, the first Socure reason code can replace the normal mapped reasons with one Socure-specific adverse-action sentence
- for machine-learning failures, the document resolver may ignore those mapped reasons and instead translate ML feature codes into approved customer-facing text

The ML translation surface is split:

- `MlFeatureCodeResolver` maps model feature codes to approved ML adverse-action lines and can also derive TransUnion TruVision reasons from embedded report features
- `FraudMlFeatureFamilyResolver` maps direct-ML fraud-model features to approved sentences
- `SocureRuleNameResolver` maps Socure identity reason codes to rule names, which can then be mapped to customer-facing adverse-action text
- `ScoreFactorCodeResolver` separately turns score-factor codes from the credit report into the bureau-specific "key factors" lines used on credit-score templates

For the direct ML path:

- if `IdentityReasonCodes` contain typed pre-resolved identity information, Compliance prefers those provider-agnostic reason lines
- otherwise it uses `MachineLearningDeclineReasons` and resolves them through `MlFeatureCodeResolver`

## 5. Choose the concrete document template

Template selection is owned by `IAdverseActionNoticeDocumentResolverWrapper`.

That wrapper selects one of two resolver implementations behind feature flag `use_newer_version_of_aan`:

- `AanDocumentResolver` is the older resolver family
- `SimplifiedAanDocumentResolver` is the newer simplified family

The high-level rules are:

- `AssessmentType.AchUnderwritingAssessmentDeclined` generates the ACH/generic decline document
- no usable credit score generates the null-credit-score template
- non-ML failures with a credit score generate the decline-reasons-plus-score-factor-codes template
- ML failures with resolved ML reasons and a credit score also generate a decline-reasons-plus-score-factor-codes template, but with ML-derived adverse-action lines
- ML failures with a credit score but without resolved ML reasons fall back to the score-factor-code-only denial template
- bureau-specific variants are selected from `creditReports.SelectedCreditBureau`

The current template families in code are the familiar `3.1`, `3.2`, and `3.3` variants plus the ACH/generic decline document:

- `3.1`: decline reasons plus score factor codes
- `3.2`: null credit score
- `3.3`: score factor codes only
- `AchUnderwritingDocument` or `GenericDeclineDocument`: ACH-specific branch

## 6. Render the PDF and store it in blob storage

`PdfGenerator` calls QuestPDF to render the selected `IGeneratableDocument`.

It then validates the bytes by checking for the expected PDF magic header before returning a stream. If the document is null or the generated bytes do not look like a PDF, Compliance throws a document-generation exception instead of silently storing bad content.

`AanDocumentRepository.SaveDocument(...)` writes the stream to blob container `adverse-action-notices`.

The blob naming model is:

- path: `<customerId>/<noticeId>.pdf`
- both the customer id and the notice id are lowercased

The `noticeId` is not always the same as the original assessment id:

- standard notices use `AssessmentId.ToString()`
- ACH underwriting notices intentionally use a new GUID so Compliance does not collide with a document already saved for the earlier decline on the same customer journey

When the write succeeds, the shared handler constructs a `NoticeInfo` containing:

- `CreatedDate`
- `NoticeId`

That `NoticeInfo` becomes the retrieval reference recorded in Cosmos.

## 7. Persist the retrieval reference and status

After the PDF is stored, the standard and direct-ML handlers append the new `NoticeInfo` to the customer's `AdverseActionNoticeReference` record in the Cosmos `AdverseActionNotices` container.

That reference store is what later powers:

- list-all-notices for a customer
- lookup-by-date
- duplicate detection for the generic paths

The standard handler may also update state in a second store:

- ACH underwriting notices update `CustomerAchUnderwritingRecord` to `AanStatus.Sent`
- enhanced-IDV status updates also exist in code, but that ingress was not the focus of this pass

## 8. Emit eligibility and sent fanout

Compliance emits two distinct kinds of downstream signal.

Eligibility fanout:

- `NotificationsManager.EmitAdverseActionNoticeEligibilityEvent(...)`
- publishes `AdverseActionNoticeEligibilityDetermined.EventHubNotification`
- increments StatsD metric `zip.compliance.aan-eligibility`

Sent fanout:

- `NotificationsManager.SendAanNotification(...)`
- publishes `AdverseActionNoticeSent.EventHubNotification`
- increments StatsD metric `zip.compliance.aan-sent`

Delivery type depends on `AssessmentType`:

- app, mobile, and ACH underwriting use `MobileApp`
- checkout purchase and checkout prequal use `CustomerPortal`

Checkout prequalification has one extra branch:

- `CheckoutPreQualNotificationsManager` publishes `CheckoutPreQualificationAanCreated`
- if Compliance cannot find the customer projection row, it falls back to a standalone download URL keyed by AAN date instead of the normal customer-portal URL

# Special Paths

## Delayed ACH path

`InitialExposureAssessmentDelayDeclined` does not create an AAN immediately.

Instead:

1. `AdverseActionNoticeHandler` converts the event to a standard command.
2. `AchUnderwritingDelayAanManager` writes or updates a `CustomerAchUnderwritingRecord` with:
   - `AchUnderWritingStatus = AchPending`
   - `AanStatus = Pending`
   - the serialized command payload and correlation id
3. The manager schedules `PendingAchUnderwritingAssessmentAan` for `EventCreatedDate + PublishDelayInMinutes`.
4. `PendingAchUnderwritingAssessmentAanHandler` later reloads the ACH record and checks:
   - the record still exists
   - `AanStatus` is still `Pending`
   - `AchUnderWritingStatus` is still `AchPending`
   - the correlation id still matches
5. Only then does it send the stored `CreateAdverseActionNotice.Command` into the normal shared handler.

This is how Compliance delays AAN generation until the ACH underwriting timeout has actually elapsed.

## ACH underwriting final decline path

`AchUnderwritingAssessmentDeclined` runs a slightly different sequence:

1. `AchUnderwritingAssessmentDeclinedHandler` first sends `CashFlowUnderwritingUpdateAan.Command` to update the ACH record status.
2. It resolves the failure type through `GetFailureRuleTypeAchUnderwriting()`, which currently always returns `RuleTypes.Credit`.
3. It creates and sends the normal `CreateAdverseActionNotice.Command`.
4. The shared handler generates the document using the ACH template branch and updates the ACH record to `AanStatus.Sent`.

## Direct machine-learning path

`MachineLearningAdverseActionDeclined` is the cleanest match to the request's description of Decision Engine-to-Compliance AAN generation.

In this path:

- the bus message already contains the relevant reasons for notice generation
- Compliance does not call the standard eligibility resolver
- Compliance does not call the standard ML reason facade
- Compliance dedupes, resolves pre-resolved identity lines when available, resolves a document, stores the PDF, records the reference, emits eligibility, and then emits the sent event using a synthetic standard command only for notification routing

This path exists so Decision Engine can send Compliance a compact, already-scored machine-learning decline payload without requiring Compliance to reconstruct the reason text from the older ingestion models.

# Storage And Retrieval Model

| Concern | Store | What is saved | Why it exists |
| --- | --- | --- | --- |
| Document bytes | Blob container `adverse-action-notices` | rendered PDF stream keyed by customer id and notice id | durable customer-facing document storage |
| Notice index | Cosmos container `AdverseActionNotices` | list of `NoticeInfo` references per customer | duplicate detection and retrieval APIs |
| Delayed ACH coordination | Cosmos container `CustomerAchUnderwritingRecords` | ACH status, pending/sent state, stored command data, correlation id | delayed AAN scheduling and ACH-specific dedupe |

Customer retrieval happens through three query handlers:

- `GetAdverseActionNoticeReferences`
- `GetAdverseActionNoticeReferenceByDate`
- `GetAdverseActionNoticeDocument`

The API controllers simply expose those handlers over:

- `GET /compliance/customers/{CustomerId}/adverseactionnotices`
- `GET /compliance/customers/{CustomerId}/adverseactionnotices/date/{Date}`
- `GET /compliance/customers/{CustomerId}/adverseactionnotice-documents/{Reference}`

# Current Caveats

- The shared standard handler recreates `AanEligibilityResolution` during `AugmentCommand(...)`. As a result, ingress-specific eligibility-reason detail does not survive intact into the final emitted eligibility event for every path.
- All `RuleTypes.None` exits in the shared standard handler are currently re-labeled as `IsNotHardDecline = true`, even when the original eligibility reason was something more specific such as non-US territory or non-WebBank eligibility.
- The delayed ACH constructor stores `AssessmentType.InitialExposureAssessmentDeclined` rather than a distinct delayed assessment type, so the downstream document and notification flow behaves like the initial-exposure branch once the delayed message is replayed.
- The standard shared handler's customer-name lookup assumes a customer projection row exists when `CustomerPersonalInfo` was not already provided. The direct-ML helper is more defensive than the standard helper.
- The direct-ML pre-resolved identity path keeps only the first fraud-model code and the first identity-provider-verification code that can be resolved into customer-facing language.
- Retroactive generation reuses the resolver and storage concepts but is a separate operational path from the live decline flows described above.

# Exact Evidence Anchors

- Runtime wiring:
  - `src/Zip.Compliance/Program.cs`
  - `src/Zip.Compliance/ServiceCollectionExtensions/AanServiceCollectionExtensions.cs`
  - `src/Zip.Compliance/ServiceCollectionExtensions/AuditLogDecoratorCollectionExtensions.cs`
- Ingress handlers:
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/InitialExposureAssessmentDeclinedHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/CheckoutPurchaseRequestDeniedHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/CheckoutPreQualificationDeniedHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/MobileAppPurchaseRequestDeclinedHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/AdverseActionNoticeHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/AchUnderwritingAssessmentDeclinedHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/AdverseActionNotices/MachineLearningAdverseActionDeclinedHandler.cs`
  - `src/Zip.Compliance/AAN/EventHandlers/PendingAchUnderwritingAssessmentAanHandler.cs`
- Shared execution:
  - `src/Zip.Compliance/AAN/Commands/CreateAdverseActionNotice.cs`
- Reason and template resolution:
  - `src/Zip.Compliance/AAN/Services/EligibilityResolver.cs`
  - `src/Zip.Compliance/AAN/Services/MappedRuleTypes.cs`
  - `src/Zip.Compliance/AAN/Services/MlFeatureCodeResolver.cs`
  - `src/Zip.Compliance/AAN/Services/FraudMlFeatureFamilyResolver.cs`
  - `src/Zip.Compliance/AAN/Services/SocureRuleNameResolver.cs`
  - `src/Zip.Compliance/AAN/Services/ScoreFactorCodeResolver.cs`
  - `src/Zip.Compliance/AAN/Services/Resolvers/AdverseActionNoticeDocumentResolverWrapper.cs`
  - `src/Zip.Compliance/AAN/Services/Resolvers/AanDocumentResolver.cs`
  - `src/Zip.Compliance/AAN/Services/Resolvers/SimplifiedAanDocumentResolver.cs`
- Storage and fanout:
  - `src/Zip.Compliance/AAN/Services/PdfGenerator.cs`
  - `src/Zip.Compliance/AAN/Services/NotificationsManager.cs`
  - `src/Zip.Compliance/AAN/Services/CheckoutPreQualNotificationsManager.cs`
  - `src/Zip.Compliance/Data/Repositories/AanDocumentRepository.cs`
  - `src/Zip.Compliance/Data/Repositories/AanReferenceRepository.cs`
  - `src/Zip.Compliance/Data/Repositories/CustomerAchUnderwritingRepository.cs`
  - `src/Zip.Compliance/AAN/Queries/*.cs`

# Related Docs

- Landing page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396561932/Compliance+System+Documentation`
- Overview page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395546162/Compliance+-+Overview`
- Reference page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395185701/Compliance+-+Reference`
- Operability guide: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396070408/Compliance+-+Operability+Guide`
- Related current-state flow map: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5250678813/CreateAdverseActionNotice+Flow+Map`
