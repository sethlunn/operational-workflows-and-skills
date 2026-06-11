# Compliance Reference

- Service or system: `decisioning-services / compliance`
- Repo path: `services/compliance`
- Exact scope: deployable inventory, HTTP surface, async interfaces, data stores, runtime entities, and exact evidence anchors as reviewed on `2026-04-21`
- Canonical owner:
  - existing service page: [Compliance](https://quadpay.atlassian.net/wiki/spaces/QD/pages/4606001162/Compliance)
  - team anchor: [Underwriting Team](https://quadpay.atlassian.net/wiki/spaces/QD/pages/4530274327/Underwriting+Team)
- Last reviewed: `2026-04-21`

# Deployables

| Name | Type | Entrypoint | Environment notes | Important links |
| --- | --- | --- | --- | --- |
| `compliance` | ASP.NET web API | `src/Zip.Compliance/Program.cs` | Production deployment `compliance` runs in `eastus-production / api`; development and sandbox variants also exist. Ingress enabled for API environments, readiness uses `/healthz`. | `azure-pipelines-compliance.yml`, `helm/compliance/` |
| `compliance-projections-orders` | background projection worker | `src/Zip.Compliance.Projections.Orders/Program.cs` | Single-replica recreate deployment; no service or ingress; production and sandbox deployments observed in Dynatrace. | `azure-pipelines-compliance-projections-orders.yml`, `helm/compliance-projections-orders/` |
| `compliance-projections-paymentplansv3` | background projection worker | `src/Zip.Compliance.Projections.PaymentPlansV3/Program.cs` | Single-replica recreate deployment; no service or ingress; production service entity plus background-activity entity observed in Dynatrace. | `azure-pipelines-compliance-projections-paymentplansv3.yml`, `helm/compliance-projections-paymentplansv3/` |
| `compliance-projections-purchaseledgers` | background projection worker | `src/Zip.Compliance.Projections.PurchaseLedgers/Program.cs` | Single-replica recreate deployment; no service or ingress; production and development deployments observed. | `azure-pipelines-compliance-projections-purchaseledgers.yml`, `helm/compliance-projections-purchaseledgers/` |
| `compliance-projections-merchantsv2` | background projection worker | `src/Zip.Compliance.Projections.MerchantsV2/Program.cs` | Single-replica recreate deployment; no service or ingress; production deployment observed clearly in Dynatrace. | `azure-pipelines-compliance-projections-merchantsv2.yml`, `helm/compliance-projections-merchantsv2/` |

# HTTP Surface

Base path from code: `/compliance`

| Verb | Path | Purpose | Auth notes | Implementation anchor |
| --- | --- | --- | --- | --- |
| `GET` | `/compliance/customers/{CustomerId}/adverseactionnotices` | List AAN references for a customer | `[Authorize]` | `AAN/AdverseActionNoticeController.cs` |
| `GET` | `/compliance/customers/{CustomerId}/adverseactionnotices/date/{Date}` | Look up AAN reference by date | `[Authorize]` | `AAN/AdverseActionNoticeController.cs` |
| `GET` | `/compliance/customers/{CustomerId}/adverseactionnotice-documents/{Reference}` | Stream AAN document from blob storage | `[Authorize]` | `AAN/AdverseActionNoticeController.cs` |
| `GET` | `/compliance/internal/customers/{CustomerId}/adverseactionnotices` | Internal list of customer AAN references | `[Authorize(Policy = "Internal")]` | `AAN/AdverseActionNoticeInternalController.cs` |
| `GET` | `/compliance/internal/customers/{CustomerId}/adverseactionnotices/date/{Date}` | Internal date lookup for AAN references | `Internal` | `AAN/AdverseActionNoticeInternalController.cs` |
| `GET` | `/compliance/internal/customers/{CustomerId}/adverseactionnotice-documents/{Reference}` | Internal AAN document retrieval | `Internal` | `AAN/AdverseActionNoticeInternalController.cs` |
| `POST` | `/compliance/bank-partner-identification` | Resolve bank partner from merchant and customer context | `Internal` | `BankPartnerIdentification/BankPartnerIdentificationController.cs` |
| `POST` | `/compliance/tila/apr/calculate` | Calculate display APR for a disclosure request | `[Authorize]` | `TILA/AprController.cs` |
| `GET` | `/compliance/tila/apr/{PaymentPlanId}` | Return APR for an existing payment plan | `[Authorize]` | `TILA/AprController.cs` |
| `POST` | `/compliance/tila` | Create or update pre-purchase TILA disclosure | `[Authorize]` | `TILA/TilaController.cs`, `TILA/Commands/CreateTilaDisclosure.cs` |
| `GET` | `/compliance/tila/{CustomerId}/{OrderId}` | List disclosure references for a customer order | `[Authorize]` | `TILA/TilaController.cs` |
| `GET` | `/compliance/tila/{CustomerId}/{OrderId}/{TilaId}` | Retrieve disclosure payload | `[Authorize]` | `TILA/TilaController.cs` |
| `GET` | `/compliance/tila/{CustomerId}/{OrderId}/{TilaId}/pdf` | Generate and stream TILA PDF | `[Authorize]` | `TILA/TilaController.cs` |
| `POST` | `/compliance/internal/tila` | Internal pre-purchase TILA creation | `Internal` | `TILA/TilaInternalController.cs` |
| `POST` | `/compliance/internal/tila/{OrderId}` | Internal post-purchase or over-auth TILA creation | `Internal` | `TILA/TilaInternalController.cs`, `TILA/Commands/CreatePostPurchaseTilaDisclosure.cs` |
| `GET` | `/compliance/internal/tila/{CustomerId}/{OrderId}` | Internal disclosure-reference lookup | `Internal` | `TILA/TilaInternalController.cs` |
| `GET` | `/compliance/internal/tila/{CustomerId}/{OrderId}/{TilaId}/pdf` | Internal TILA PDF retrieval | `Internal` | `TILA/TilaInternalController.cs` |
| `GET` | `/compliance/healthz` | readiness and liveness endpoint | no controller auth | `Program.cs` |

# Async Interfaces

| Direction | Name | Purpose | Producer or consumer | Implementation anchor |
| --- | --- | --- | --- | --- |
| Inbound | Queue `service-compliance` | Primary Azure Service Bus receive queue for API-hosted handlers | Consumer in `Zip.Compliance` | `Program.cs` |
| Inbound | Topic `integration-events` events `CheckoutPreQualificationAanCreated`, `TilaGenerated` | Feed AAN and TILA event handlers from shared integration topic | Consumer in `Zip.Compliance` | `Program.cs` |
| Inbound | Topic `compliance-events` event `PendingAchUnderwritingAssessmentAan` | Feed ACH-related AAN handling | Consumer in `Zip.Compliance` | `Program.cs` |
| Inbound | Subscription `AdverseActionNoticeEventHandlers` on `compliance-events` | Consume denial and AAN-generation events such as `CheckoutPreQualificationDenied`, `GenerateRetroactiveAan`, `AchUnderwritingAssessmentDeclined`, `MachineLearningAdverseActionDeclined`, and related denial events | Consumer in `Zip.Compliance` | `ServiceCollectionExtensions/AanServiceCollectionExtensions.cs` |
| Inbound | Subscription `TilaEventHandlers` on `compliance-events` | Consume `PaymentPlanAmountIncreasedEvent` for TILA and payment-plan updates | Consumer in `Zip.Compliance` | `ServiceCollectionExtensions/AanServiceCollectionExtensions.cs` |
| Outbound | Topic `service-analytics` event `TilaCreated` | Emit analytics event for disclosure creation | Producer in `Zip.Compliance` | `Program.cs`, `TILA/IntegrationEvents/` |
| Outbound | Event Hub connection `EventHubDecisionEngine` | Best-effort audit history publication | Producer in `Zip.Compliance` | `EventHubs/EventHubsServiceCollectionExtensions.cs` |
| Inbound | EventStore source `merchant-*` | Populate merchants projection read model | Consumer in `Zip.Compliance.Projections.MerchantsV2` | `Zip.Compliance.Projections.MerchantsV2/Program.cs` |
| Inbound | EventStore category `order` | Populate orders projection read model | Consumer in `Zip.Compliance.Projections.Orders` | `Zip.Compliance.Projections.Orders/Program.cs` |
| Inbound | EventStore source `paymentPlan-*` | Populate payment-plans projection read model; background-activity telemetry also appears in Dynatrace | Consumer in `Zip.Compliance.Projections.PaymentPlansV3` | `Zip.Compliance.Projections.PaymentPlansV3/Program.cs` |
| Inbound | EventStore category `purchaseLedger` | Populate purchase-ledger projection read model | Consumer in `Zip.Compliance.Projections.PurchaseLedgers` | `Zip.Compliance.Projections.PurchaseLedgers/Program.cs` |

# Data Stores And Schemas

- Data stores used:
  - SQL Server connection `DecisioningCriticalSqlServer`
  - Cosmos DB database `compliance`
  - blob storage connection `DecisioningComplianceBlobStorage`
  - Event Hubs connection `EventHubDecisionEngine`
- Important tables, collections, or documents:
  - SQL schema `DecisionEngineCustomerProjection`
  - SQL schema `ComplianceMerchantProjectionsV2`
  - SQL schema `ComplianceOrdersProjections`
  - SQL schema `CompliancePaymentPlanProjectionsV3`
  - SQL schema `CompliancePurchaseLedgerProjections`
  - Cosmos containers `AdverseActionNotices`, `CustomerAchUnderwritingRecords`, and `EnhancedIdvRecords`
  - blob containers `tila-disclosures` and `adverse-action-notices`
- Important schema or contract anchors:
  - `Data/CustomerDbContext.cs`
  - `Data/ComplianceMerchantProjectionsDbContext.cs`
  - `Data/OrderDbContext.cs`
  - `Data/PaymentPlansV3DbContext.cs`
  - `Data/PurchaseLedgerDbContext.cs`
  - `ServiceCollectionExtensions/CosmosExtensions.cs`
  - `Data/BlobSettings.cs`
  - `Data/Repositories/*.cs`
- Data caveats:
  - several API read paths depend on projection freshness rather than source-of-truth aggregates
  - Cosmos and blob container names differ in casing between code constants and environment config surfaces
  - `DecisioningComplianceSqlServer` appears in config, but the API read models inspected here are wired from `DecisioningCriticalSqlServer`

# Runtime Entities

- Dynatrace or platform entities:
  - production API services: `SERVICE-CB677C879B93D275`, `SERVICE-543A858BE69CF723`
  - production API deployment: `compliance` on `eastus-production / api`
  - production worker services:
    - `SERVICE-36FA4D6FC506995B` for `compliance-projections-orders`
    - `SERVICE-38BE9026B1F02204` for `compliance-projections-paymentplansv3`
    - `SERVICE-7BACAEE43A66A01C` and `SERVICE-31690706F53DD3E1` for `compliance-projections-purchaseledgers`
    - deployment-only visibility was clearest for `compliance-projections-merchantsv2`
  - production payment-plan background activity service: `SERVICE-A2F0FA64FC0F7845`
- Environment mapping:
  - API production: `eastus-production / api`
  - API development: `eastus-dev / master` and `zip-aks-cluster-dev-02 / master`
  - sandbox: `eastus-sbx / sandbox`
  - worker production deployments observed in `eastus-production / api`
- Known naming caveats:
  - API service surfaces as `compliance-primary` in Dynatrace while Helm service name is `compliance`
  - machine-learning gateway dependency appears as both `machinelearning-gateway-primary` and `QuadPay.ProviderGateways.MachineLearning`

# Config, Flags, And Ownership

- Key config surfaces:
  - `ConnectionStrings:DecisioningCriticalSqlServer`
  - `ConnectionStrings:DecisioningComplianceBlobStorage`
  - `ConnectionStrings:EventHubDecisionEngine`
  - `ComplianceCosmos`
  - `BlobSettings`
  - `RiskModels`
  - `CreditBureauGateway`
  - `MachineLearningGateway`
  - `LabSettings`
  - `Jwt`
  - `DecisionEngine`
  - `QpawSettings`
- Feature flags:
  - `useaprcalculation`
  - `compliance_enable_lab_ml_reason_codes`
  - `evaluate_prepurchase_tila_eligibility`
  - `use_newer_version_of_aan`
  - `machinelearningengineering_bypass`
  - `use_new_uccc_tila_wording_5_sep_2025`
  - `enable_socure_dynamic_features`
  - `enable_cko_socure_dynamic_features`
- Dashboards, alerts, or SLO anchors:
  - `services/compliance/service.json`
  - current endpoint-analysis page: [Compliance Endpoint Traffic Analysis](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5266636805/Compliance+Endpoint+Traffic+Analysis)
- Confluence, repo, or service-definition links:
  - [Compliance](https://quadpay.atlassian.net/wiki/spaces/QD/pages/4606001162/Compliance)
  - [Operation Clarity - Compliance Service Codebase Documentation & Refactor](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5250449424/Operation+Clarity+Compliance+Service+Codebase+Documentation+Refactor)
  - `services/compliance/service.json`

# Exact Evidence Anchors

- Important files inspected:
  - `services/compliance/service.json`
  - `services/compliance/README.md`
  - `services/compliance/azure-pipelines-*.yml`
  - `services/compliance/helm/**/values/*.yaml`
  - `services/compliance/src/Zip.Compliance/Program.cs`
  - `services/compliance/src/Zip.Compliance/**/*Controller.cs`
  - `services/compliance/src/Zip.Compliance/ServiceCollectionExtensions/*.cs`
  - `services/compliance/src/Zip.Compliance/Data/*.cs`
  - `services/compliance/src/Zip.Compliance.Projections.*/Program.cs`
  - `services/compliance/src/Zip.Compliance.Projections.*/Projection*.cs`
- Important queries used:

```text
smartscapeNodes "SERVICE" | filter id == toSmartscapeId("SERVICE-CB677C879B93D275") or id == toSmartscapeId("SERVICE-543A858BE69CF723")
smartscapeEdges "*" | filter source_id == toSmartscapeId("SERVICE-CB677C879B93D275") or target_id == toSmartscapeId("SERVICE-CB677C879B93D275")
smartscapeNodes "SERVICE" | filter id == toSmartscapeId("SERVICE-67C0A37B04791199") or id == toSmartscapeId("SERVICE-6A371B00F88E3673") or id == toSmartscapeId("SERVICE-ABF908062BB35ADC") or id == toSmartscapeId("SERVICE-C9A4BAB9D4320571") or id == toSmartscapeId("SERVICE-D50BFDE34543CBF9") or id == toSmartscapeId("SERVICE-7EA564965BD799AD") or id == toSmartscapeId("SERVICE-F4F7F2376D687A69")
timeseries from: now()-24h, requests = sum(dt.service.request.count, scalar: true), failures = sum(dt.service.request.failure_count, scalar: true), avg_response = avg(dt.service.request.response_time, scalar: true), by:{endpoint.name}, filter: dt.entity.service == "SERVICE-CB677C879B93D275" and endpoint.name != "/healthz" and endpoint.name != "GET /compliancepolicies.inc.php" and endpoint.name != "CreateBatchAsync" and endpoint.name != "Process" and endpoint.name != "Send" and endpoint.name != "SendAsync" and endpoint.name != "ProcessMessages"
smartscapeNodes "K8S_DEPLOYMENT" | filter name == "compliance"
smartscapeNodes "K8S_DEPLOYMENT" | filter name == "compliance-projections-orders" or name == "compliance-projections-paymentplansv3" or name == "compliance-projections-purchaseledgers" or name == "compliance-projections-merchantsv2"
```

- Important external links:
  - [Compliance service page](https://quadpay.atlassian.net/wiki/spaces/QD/pages/4606001162/Compliance)
  - [Compliance Endpoint Traffic Analysis](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5266636805/Compliance+Endpoint+Traffic+Analysis)
  - [Operation Clarity - Compliance Service Codebase Documentation & Refactor](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5250449424/Operation+Clarity+Compliance+Service+Codebase+Documentation+Refactor)

# Related Docs

- Landing page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396561932/Compliance+System+Documentation`
- Overview page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395546162/Compliance+-+Overview`
- Operability guide: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396070408/Compliance+-+Operability+Guide`
- Tutorial:
  - none yet
