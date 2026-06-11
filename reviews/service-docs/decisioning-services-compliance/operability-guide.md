# Compliance Operability Guide

- Service or system: `decisioning-services / compliance`
- Audience: responders, service owners, and engineers validating health or tracing production behavior
- Environments covered:
  - production first
  - development and sandbox only where they help disambiguate runtime entities
- Exact scope: health checks, rollout validation, request and event tracing, dependency checks, and common failure modes reviewed on `2026-04-21`
- Last reviewed: `2026-04-21`

# Common Tasks

- Validate service health:
  - confirm production API service entities `SERVICE-CB677C879B93D275` and `SERVICE-543A858BE69CF723`
  - check last-24-hour request, failure, and average response-time rollups on `SERVICE-CB677C879B93D275`
  - confirm projection workers exist in `eastus-production / api` and that their deployments are recent and available
- Validate a deployment or rollout:
  - correlate the deployment version from Dynatrace or Kubernetes to Helm release `compliance-qp-api`
  - compare `POST /tila`, `POST /tila/apr/calculate`, `GET /customers/{CustomerId}/adverseactionnotices`, and `GET /tila/{CustomerId}/{OrderId}` before and after the rollout window
  - for workers, confirm EventStore-driven deployments restarted cleanly and did not lose projection freshness
- Trace a request, event, or identifier:
  - for TILA work, start with `CustomerId`, `OrderId`, and `TilaId`
  - for AAN work, start with `CustomerId` plus document `Reference`
  - for bank-partner issues, start with `CustomerId`, `MerchantId`, and whether projection fallback was used
  - for projection issues, identify the affected aggregate category first: merchant, order, payment plan, or purchase ledger
- Confirm dependency health:
  - if TILA or AAN behavior looks wrong, check projection freshness before blaming external dependencies
  - if AAN reason-code behavior is wrong, confirm whether the path hit LAB, risk model, credit-bureau gateway, or machine-learning gateway
  - if API responses are healthy but audit consumers are missing data, check Event Hub audit writes separately

# Health Checks And Signals

- Primary health indicators:
  - last 24 hours ending about `2026-04-21 15:10 UTC` on production API service `SERVICE-CB677C879B93D275`:
    - `POST /tila/apr/calculate`: `445,993` requests, `0` failures, average response time about `4.25ms`
    - `POST /tila`: `241,220` requests, `1,814` failures, average response time about `75.18ms`
    - `GET /customers/{CustomerId}/adverseactionnotices`: `205,480` requests, `0` failures, average response time about `10.85ms`
    - `GET /tila/{CustomerId}/{OrderId}`: `119,470` requests, `0` failures, average response time about `18.53ms`
  - lower-volume but materially slower document routes:
    - `GET /customers/{CustomerId}/adverseactionnotice-documents/{Reference}`: about `235.61ms`
    - `GET /tila/{CustomerId}/{OrderId}/{TilaId}/pdf`: about `283.80ms`
- Important dashboards or alerts:
  - Dynatrace service view for `SERVICE-CB677C879B93D275`
  - service-definition SLO block in `services/compliance/service.json`
  - existing analysis page: [Compliance Endpoint Traffic Analysis](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5266636805/Compliance+Endpoint+Traffic+Analysis)
- Important Dynatrace entities:
  - production API services: `SERVICE-CB677C879B93D275`, `SERVICE-543A858BE69CF723`
  - production worker services: `SERVICE-36FA4D6FC506995B`, `SERVICE-38BE9026B1F02204`, `SERVICE-7BACAEE43A66A01C`, `SERVICE-31690706F53DD3E1`
  - production payment-plan background service: `SERVICE-A2F0FA64FC0F7845`
- Exact queries or signal anchors:

```text
timeseries from: now()-24h, requests = sum(dt.service.request.count, scalar: true), failures = sum(dt.service.request.failure_count, scalar: true), avg_response = avg(dt.service.request.response_time, scalar: true), by:{endpoint.name}, filter: dt.entity.service == "SERVICE-CB677C879B93D275" and endpoint.name != "/healthz" and endpoint.name != "GET /compliancepolicies.inc.php" and endpoint.name != "CreateBatchAsync" and endpoint.name != "Process" and endpoint.name != "Send" and endpoint.name != "SendAsync" and endpoint.name != "ProcessMessages"
smartscapeEdges "*" | filter source_id == toSmartscapeId("SERVICE-CB677C879B93D275") or target_id == toSmartscapeId("SERVICE-CB677C879B93D275")
smartscapeNodes "K8S_DEPLOYMENT" | filter name == "compliance"
smartscapeNodes "K8S_DEPLOYMENT" | filter name == "compliance-projections-orders" or name == "compliance-projections-paymentplansv3" or name == "compliance-projections-purchaseledgers" or name == "compliance-projections-merchantsv2"
```

# Common Failure Modes

- Failure mode:
  - how it presents
    - `POST /tila` returns `400` or `500`, or caller sees missing disclosure creation on a hot path
  - first checks
    - confirm the request is hitting `CreateTilaDisclosure`
    - inspect pre-purchase eligibility decisions and token-deserialization failures
    - confirm projection and blob dependencies are healthy before blaming controller auth
  - strongest confirming signal
    - production request failures concentrated on endpoint `tila`

- Failure mode:
  - how it presents
    - TILA PDF or AAN document retrieval is slow or intermittently unavailable
  - first checks
    - inspect blob-container access for `tila-disclosures` or `adverse-action-notices`
    - compare current latency against the higher SLO overrides for document routes
    - confirm the requested `TilaId` or AAN `Reference` exists before treating it as an infra failure
  - strongest confirming signal
    - elevated latency on `GET /tila/{CustomerId}/{OrderId}/{TilaId}/pdf` or `GET /customers/{CustomerId}/adverseactionnotice-documents/{Reference}` without broader API degradation

- Failure mode:
  - how it presents
    - AAN generation or reason codes look wrong after denial events
  - first checks
    - confirm `compliance-events` subscriptions are active
    - inspect feature flags such as `compliance_enable_lab_ml_reason_codes`, `machinelearningengineering_bypass`, and `use_newer_version_of_aan`
    - determine whether the path should have called risk model, LAB, credit-bureau gateway, or machine-learning gateway
  - strongest confirming signal
    - mismatch between denial event handling and the configured feature-flag path, or downstream dependency failure on one of the reason-code providers

- Failure mode:
  - how it presents
    - bank-partner identification disagrees with merchant configuration or decisioning expectations
  - first checks
    - inspect `IdentifyBankPartner.Command`
    - confirm merchant projections are current and whether `ProjectionFallback` was enabled
    - verify merchant and customer geography inputs
  - strongest confirming signal
    - incorrect or stale merchant projection data rather than controller failure

- Failure mode:
  - how it presents
    - disclosures or compliance reads lag behind source-of-truth changes
  - first checks
    - determine which projection worker owns the stale field
    - confirm the relevant deployment is up in `eastus-production / api`
    - inspect EventStore subscription source for the correct stream category or prefix
  - strongest confirming signal
    - worker deployment healthy but projection data stale, or worker deployment missing after rollout

# Debugging And Investigation Steps

- Step 1:
  - identify the exact surface first:
    - TILA create or retrieve
    - AAN create or retrieve
    - bank-partner identification
    - projection freshness

- Step 2:
  - validate the runtime entity:
    - API production traffic and failures: `SERVICE-CB677C879B93D275`
    - API current Smartscape surface: `SERVICE-543A858BE69CF723`
    - worker services: `SERVICE-36FA4D6FC506995B`, `SERVICE-38BE9026B1F02204`, `SERVICE-7BACAEE43A66A01C`, `SERVICE-31690706F53DD3E1`

- Step 3:
  - branch by storage path:
    - projection-backed reads: check SQL projection schemas first
    - disclosure and notice document reads: check blob containers first
    - AAN history or ACH records: check Cosmos containers first

- Step 4:
  - branch by async trigger:
    - if the issue began from a denial or ACH event, inspect `compliance-events` subscriptions
    - if the issue is payment-plan-related, inspect both API behavior and `PaymentPlansV3` projection freshness
    - if the issue is a stale merchant or order field, inspect the matching projection worker rather than the API route alone

- Step 5:
  - confirm downstream dependencies only after local state checks:
    - risk model
    - credit-bureau gateway
    - machine-learning gateway
    - LAB
    - Event Hubs

- Step 6:
  - carry forward exact evidence:
    - endpoint or event name
    - environment
    - service or deployment id
    - time window
    - request and failure counts
    - customer, order, payment-plan, or reference identifiers used in the trace

# Escalation And Hand-Off

- When to escalate:
  - when `POST /tila` failures continue after ruling out request validation and projection freshness
  - when projection workers are healthy but downstream systems are clearly returning bad or missing data
  - when machine-learning, risk-model, or credit-bureau failures are the primary cause
  - when multiple compliance deployables show simultaneous issues after a rollout
- Which evidence to carry forward:
  - exact endpoint or topic/subscription
  - exact Dynatrace entity or deployment id
  - exact time window
  - request, failure, and latency observations
  - relevant customer, order, payment-plan, merchant, or document identifiers
  - feature-flag state when AAN or reason-code logic is involved
- Which related teams or services may need to be involved:
  - Decision Engine
  - Orders
  - Checkout or Frontend orchestration
  - Machine Learning Gateway owners
  - Risk Model owners
  - Credit Bureau Gateway owners
  - platform owners for cluster, pod, or storage issues

# Caveats

- What this guide can answer directly:
  - current API hot paths
  - deployment and service-entity anchors
  - likely local storage and projection dependencies
  - topology-confirmed upstream and downstream services
- What requires deeper investigation:
  - business correctness of disclosure or notice text
  - semantic correctness of risk-model or ML outputs
  - caller-side misuse of internal vs public endpoints
- Known telemetry gaps:
  - worker telemetry is not uniform; `MerchantsV2` was clearest as a deployment rather than a service entity
  - API response-time values were interpreted as microseconds for human-readable milliseconds
  - `decisioning-critical` and other background buckets appear in service metrics but are not controller-defined routes

# Related Docs

- Landing page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396561932/Compliance+System+Documentation`
- Overview page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395546162/Compliance+-+Overview`
- Reference page: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395185701/Compliance+-+Reference`
- Tutorial:
  - none yet
