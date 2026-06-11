# Compliance Overview

- Service or system: `decisioning-services / compliance`
- Repo path: `services/compliance`
- Exact scope: main ASP.NET API plus `Orders`, `PaymentPlansV3`, `PurchaseLedgers`, and `MerchantsV2` projection workers; code-backed runtime shape; production telemetry and topology observed on `2026-04-21`
- Audience: engineers and responders who need the service boundary and runtime shape before debugging, refactoring, or extending the system
- Last reviewed: `2026-04-21`
- Primary conclusion: Compliance is a document-and-read-model service on the decisioning path. It does not make the underlying credit decision, but it turns decisioning outcomes into customer-facing TILA and adverse-action artifacts, resolves bank-partner context, and keeps several SQL-backed projection models current from EventStore and bus signals. In production over the last 24 hours ending about `2026-04-21 15:10 UTC`, the busiest business routes were `POST /tila/apr/calculate` (`445,993` requests), `POST /tila` (`241,220` requests, `1,814` failed requests), `GET /customers/{CustomerId}/adverseactionnotices` (`205,480` requests), and `GET /tila/{CustomerId}/{OrderId}` (`119,470` requests).

# What This Service Does

- Primary responsibility:
  - generate and retrieve Truth in Lending Act disclosures
  - generate, store, retrieve, and notify on adverse action notices
  - resolve bank-partner selection for decisioning and disclosure flows
  - maintain compliance-oriented SQL projections for merchants, orders, payment plans, and purchase ledgers
- Important boundaries:
  - this service does not own the underlying lending or underwriting decision
  - it does not own the source-of-truth order, customer, or merchant aggregates
  - it does not own machine-learning model execution or credit-bureau gateway implementation
- What this service explicitly does not own:
  - caller orchestration in `decision-engine`, `checkout-orchestration`, `orders`, or frontend orchestration
  - primary customer, order, and payment-plan state creation
  - ML or risk-model business logic beyond calling those systems for supporting data

# Runtime Shape

- Deployables or applications:
  - ASP.NET application `Zip.Compliance` with base path `/compliance`
  - worker `Zip.Compliance.Projections.Orders`
  - worker `Zip.Compliance.Projections.PaymentPlansV3`
  - worker `Zip.Compliance.Projections.PurchaseLedgers`
  - worker `Zip.Compliance.Projections.MerchantsV2`
- Ingress types:
  - authenticated HTTP controller routes for public and internal TILA, AAN, and bank-partner flows
  - Azure Service Bus receive queue `service-compliance`
  - Azure Service Bus topic subscriptions on `compliance-events`
  - EventStore-backed projection subscriptions for merchant, order, payment-plan, and purchase-ledger streams
- Data stores:
  - SQL read models on `DecisioningCriticalSqlServer`
  - Cosmos DB database `compliance`
  - blob containers `tila-disclosures` and `adverse-action-notices`
  - Event Hubs audit output through connection `EventHubDecisionEngine`
- Major outbound dependencies:
  - `risk-model-2`
  - `experian-gateway`
  - `machinelearning-gateway`
  - LAB and TransUnion ML endpoints
  - Event Hubs audit writer
  - Auth0 authority and internal-auth handlers
  - Optimizely feature flags
- Ownership or on-call notes:
  - service definition anchor: [Compliance](https://quadpay.atlassian.net/wiki/spaces/QD/pages/4606001162/Compliance)
  - this doc set is published under a new Decisioning Services documentation root in Operational Analysis

# Main Flows

- Primary user or request flow:
  - an authenticated caller hits `/compliance/tila`, `/compliance/tila/apr/*`, `/compliance/customers/*/adverseactionnotices`, or `/compliance/bank-partner-identification`
  - the controller forwards to MediatR commands or queries
  - handlers combine request payloads with projection data, Cosmos records, blob storage, and downstream dependency calls
  - the API returns disclosures, references, PDFs, or bank-partner selection data
- Primary async or event-driven flow:
  - the API receives integration messages on `service-compliance` and `compliance-events`
  - AAN and TILA handlers react to denial, ACH, prequalification, or payment-plan-change events
  - projection workers tail EventStore streams and update SQL read models consumed by the API
  - the API emits `TilaCreated` to `service-analytics` and writes audit history to Event Hubs
- Important background or batch flow:
  - `Orders`, `PaymentPlansV3`, `PurchaseLedgers`, and `MerchantsV2` workers rebuild or maintain SQL projections from EventStore categories and stream prefixes
- Diagram source files:
  - `./diagrams/compliance-system-context.mmd`
  - `./diagrams/compliance-projection-topology.mmd`
- Rendered diagram snapshots:
  - `./diagrams/rendered/compliance-system-context.svg`
  - `./diagrams/rendered/compliance-projection-topology.svg`
  - current publication path does not attach local SVG files automatically; use the repo-managed source set for the rendered assets

# Dependency Model

- Upstream callers or producers:
  - Smartscape calls into production `compliance-primary` currently come from `orders-primary`, `checkout-orchestration-primary`, `decision-engine-primary`, `frontend-orchestration-primary`, and `QuadPay.FrontEndOrchestration`
  - Azure Service Bus topics feed denial, ACH, and payment-plan events into Compliance handlers
  - EventStore feeds the four projection workers
- Downstream services or providers:
  - `machinelearning-gateway-primary` and `QuadPay.ProviderGateways.MachineLearning` from current topology
  - risk model and credit-bureau gateway clients proved directly from code
  - LAB and TransUnion ML endpoints proved directly from code
  - Azure Blob Storage, Cosmos DB, SQL Server, and Event Hubs
- Shared platforms or infrastructure:
  - production deployment `compliance` in `eastus-production / api`
  - production worker deployments in `eastus-production / api`
  - development and sandbox variants also exist in `eastus-dev / master`, `zip-aks-cluster-dev-02 / master`, and `eastus-sbx / sandbox`
  - Key Vault file mounts, OTEL export, StatsD, Serilog, and Dynatrace
- Dependency caveats:
  - topology shows two active production service ids for `compliance-primary`: `SERVICE-CB677C879B93D275` and `SERVICE-543A858BE69CF723`
  - machine-learning gateway appears in both v1 and v2 service-detection forms; the code confirms it is one logical dependency
  - some worker telemetry is clearer at the deployment level than the service-entity level

# Key Data And Contracts

- Most important domain entities:
  - `CreateTilaDisclosure.Command` and `CreatePostPurchaseTilaDisclosure.Command`
  - `GetTilaDisclosure*` and `GetAdverseActionNotice*` query models
  - `AdverseActionNoticeReference`
  - `CustomerAchUnderwritingRecord`
  - `EnhancedIdvRecord`
  - projection read models for `Merchant`, `Order`, `PaymentPlan`, and `PurchaseLedger`
- Most important external contracts:
  - Azure Service Bus routes on `integration-events`, `compliance-events`, and `service-analytics`
  - Refit clients for risk model, credit-bureau gateway, LAB, and machine-learning gateway
  - Event Hub audit writer on `EventHubDecisionEngine`
  - SQL projection schemas consumed by API repositories
- Where to look for exact schemas or inventories:
  - HTTP routes: `src/Zip.Compliance/**/*Controller.cs`
  - storage and SQL schemas: `src/Zip.Compliance/Data/*.cs`
  - async subscriptions and handlers: `src/Zip.Compliance/Program.cs`, `src/Zip.Compliance/ServiceCollectionExtensions/AanServiceCollectionExtensions.cs`
  - projection workers: `src/Zip.Compliance.Projections.*/Program.cs` and `src/Zip.Compliance.Projections.*/Projection*.cs`

# Operational Shape

- Important SLOs or user-visible expectations:
  - availability target: `99.95`
  - default latency target: `100ms`
  - higher latency thresholds are explicitly allowed for TILA creation and PDF/document retrieval routes
- Most important telemetry surfaces:
  - production API service ids: `SERVICE-CB677C879B93D275` and `SERVICE-543A858BE69CF723`
  - production projection worker services: `SERVICE-36FA4D6FC506995B` (`orders`), `SERVICE-38BE9026B1F02204` (`paymentplansv3`), `SERVICE-7BACAEE43A66A01C` and `SERVICE-31690706F53DD3E1` (`purchaseledgers`)
  - production background-activity service for payment-plan projection: `SERVICE-A2F0FA64FC0F7845`
  - production deployment anchors:
    - `compliance` on `eastus-production / api`
    - `compliance-projections-paymentplansv3` on `eastus-production / api`
    - `compliance-projections-merchantsv2` on `eastus-production / api`
- Most common failure modes at a high level:
  - `POST /tila` returning failed requests on a high-volume path
  - TILA or AAN PDF/document retrieval latency driven by blob-storage fetch and generation work
  - stale projection data causing downstream document or bank-partner mismatches
  - denial or ACH event handling drift when `compliance-events` subscriptions or feature-flag paths diverge

# Caveats

- Known ambiguity:
  - current production telemetry shows overlapping `compliance-primary` service ids; endpoint counts below use `SERVICE-CB677C879B93D275` because that entity still carries the active request metrics
  - Dynatrace response-time values were interpreted as microseconds, so `75,182` was treated as about `75.18ms`
  - `decisioning-critical` still appears as a high-volume endpoint-like metric series, but it is not a controller-defined Compliance route
- Partial coverage:
  - this pass did not walk every Operation Clarity child page or every repository implementation
  - worker service-entity coverage is uneven; for `MerchantsV2`, the deployment evidence was clearer than a first-class service entity
- Out-of-scope areas:
  - business-policy correctness of APR, TILA, or AAN text
  - end-to-end caller behavior outside the Compliance boundary
  - broader product/process documentation beyond this service surface

# Related Docs

- Landing page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396561932/Compliance+System+Documentation`
- Reference page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395185701/Compliance+-+Reference`
- Operability guide: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396070408/Compliance+-+Operability+Guide`
- Tutorial:
  - none yet
