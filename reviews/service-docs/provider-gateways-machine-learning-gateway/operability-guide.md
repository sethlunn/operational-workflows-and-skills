# Machine Learning Gateway Operability Guide

- Service or system: `provider-gateways / machine-learning-gateway`
- Audience: responders, service owners, and engineers validating health or tracing failures
- Environments covered:
  - production first
  - development runtime mapping included where it helps disambiguate Dynatrace entities
- Exact scope: health checks, rollout validation, request tracing, dependency checks, and common failure modes for the gateway as reviewed on `2026-04-08`
- Last reviewed: `2026-04-08`

# Common Tasks

- Validate service health:
  - confirm the production Dynatrace entity is `SERVICE-F4F7F2376D687A69`
  - check 24-hour request volume, failures, and average response time
  - confirm there are no active or recent service-scoped Dynatrace problems
- Validate a deployment or rollout:
  - correlate the service to workload `machinelearning-gateway-primary` in `eastus-production/api`
  - compare request failures and average response time before and after the rollout window
  - check canary thresholds from Helm:
    - `p50`: `150`
    - `p99`: `8000`
- Trace a request, event, or identifier:
  - start from the concrete endpoint family
  - if the route is `POST /machinelearning-gateway/predictions`, capture both the HTTP status and the response-body `status`, attempt counts, and per-model attempts before assuming total success or failure
  - if the route is one of the direct legacy prediction or reason-code routes, jump straight to the corresponding MediatR handler
- Confirm dependency health:
  - for direct prediction routes, verify downstream Databricks endpoint behavior
  - for `POST /machinelearning-gateway/predictions`, check which selected model path failed before assuming the whole gateway is down
  - also check Event Hubs audit write behavior when the request itself appears healthy

# Health Checks And Signals

- Primary health indicators:
  - last 24 hours ending about `2026-04-08 13:00 UTC`:
    - total requests: `901,266`
    - failed requests: `11`
    - average response time: about `56.12ms`
  - all observed failed production requests in that window were `500` responses on `POST /machinelearning-gateway/predictions`
  - HTTP failure counts understate `RunPredictions` degradation because the route can also return `200 OK` with top-level `Failed` or `PartiallySucceeded`
- Important dashboards or alerts:
  - broader Dynatrace service view for `SERVICE-F4F7F2376D687A69`
  - provider-gateways service-definition SLO block in `service.json`
  - production canary thresholds in Helm values
- Important Dynatrace entities:
  - production service: `SERVICE-F4F7F2376D687A69`
  - development service: `SERVICE-40FFDB56A531E988`
  - production process group: `PROCESS_GROUP-D5E2EC6E18D78A77`
- Exact queries or signal anchors:

```text
timeseries { request_count = sum(dt.service.request.count, scalar: true), failure_count = sum(dt.service.request.failure_count, scalar: true), avg_response_time = avg(dt.service.request.response_time, scalar: true) }, filter:{dt.entity.service == "SERVICE-F4F7F2376D687A69"}, from: now()-24h
timeseries { request_count = sum(dt.service.request.count, scalar: true) }, by:{endpoint.name}, filter:{dt.entity.service == "SERVICE-F4F7F2376D687A69"}, from: now()-24h | sort request_count desc | limit 5
timeseries { failed_request_count = sum(dt.service.request.count, scalar: true) }, by:{endpoint.name, http.response.status_code}, filter:{dt.entity.service == "SERVICE-F4F7F2376D687A69" and failed == true}, from: now()-24h | sort failed_request_count desc | limit 20
```

# Common Failure Modes

- Failure mode:
  - how it presents:
    - `POST /machinelearning-gateway/predictions` returns `500`
    - production failure volume is small but currently concentrated entirely on this route
  - first checks:
    - run the failed-request DQL filtered to `SERVICE-F4F7F2376D687A69`
    - inspect `PredictionModelStrategyFactory`, `RunPredictionsCommandExtensions`, and `ConcurrentFanoutExecutor`
    - check whether the request shape should have selected Unified NCM V1 only or Unified NCM V1 plus Global Return Customer
  - strongest confirming signal:
    - Dynatrace shows `POST predictions` as the failing endpoint with `500` status

- Failure mode:
  - how it presents:
    - `POST /machinelearning-gateway/predictions` returns `200 OK` but the body `status` is `Failed` or `PartiallySucceeded`
    - callers describe bad model outcomes even though HTTP failure metrics stay low
  - first checks:
    - inspect the response-body `attempted_count`, `succeeded_count`, `failed_count`, `timed_out_count`, and `attempts`
    - map the request to the live selection rules before debugging the wrong model path
    - inspect `UnifiedNCMV1PredictionModel`, `GlobalReturnCustomerPredictionModel`, and `MachineLearningGatewayResultFactory`
    - check for unified-path request-build problems such as missing or conflicting provider blob-path mappings
  - strongest confirming signal:
    - the response payload shows mixed or failed model attempts while the request itself still returned `200 OK`

- Failure mode:
  - how it presents:
    - prediction routes slow down or start timing out without a clear code change in the gateway itself
  - first checks:
    - compare route latency against the service-definition `300ms` default threshold
    - inspect `ConcurrentFanoutExecutor`, `UnifiedNCMV1PredictionModel`, and `GlobalReturnCustomerPredictionModel`
    - distinguish the unified client path from the decorated global-return-customer path before assuming one shared timeout layer
    - inspect log messages for Databricks call failures and timeout warnings
  - strongest confirming signal:
    - Databricks-call error logs, slow route telemetry, or repeated failed attempts on one model path rather than an executor-level timeout budget breach

- Failure mode:
  - how it presents:
    - `POST /machinelearning-gateway/predictions` succeeds, but BI/audit consumers complain about missing audit events
  - first checks:
    - inspect logs for `Failed to write audit log for MachineLearningRunPredictionsCompleted`
    - inspect Event Hub configuration and Event Hub resilience settings
  - strongest confirming signal:
    - request success with matching audit-write error log, because audit publication is wrapped in a catch and does not fail the API response

- Failure mode:
  - how it presents:
    - callers receive `401` or `403` even though the route exists
  - first checks:
    - confirm the caller is using the internal audience required by the `Internal` authorization policy
    - inspect JWT authority and audience configuration
  - strongest confirming signal:
    - authentication/authorization failure before the MediatR handler is reached

# Debugging And Investigation Steps

- Step 1:
  - identify the exact route, environment, and time window
  - disambiguate the service entity:
    - production: `SERVICE-F4F7F2376D687A69`
    - development: `SERVICE-40FFDB56A531E988`
- Step 2:
  - branch by route family
  - if the route is `POST /machinelearning-gateway/predictions`, inspect:
    - the HTTP status plus top-level response `status`
    - `attempted_count`, `succeeded_count`, `failed_count`, `timed_out_count`, and `attempts`
    - `RunPredictions.cs`
    - `RunPredictionsCommandExtensions.cs`
    - `PredictionModelStrategyFactory.cs`
    - `ConcurrentFanoutExecutor.cs`
  - if the route is a direct legacy or challenger endpoint, inspect the matching handler in `MachineLearningEngineering/Gateway/Commands/` or `MachineLearningEngineering/Queries/`
- Step 3:
  - map the request to the live selection rules
  - app + `merchant_id` + TransUnion blob selects Unified NCM V1 plus Global Return Customer
  - checkout existing-customer selects Unified NCM V1 plus Global Return Customer
  - checkout new-customer + `merchant_id` + TransUnion blob currently still selects Unified NCM V1 only
  - default fallback selects Unified NCM V1 only
  - no current live `RunPredictions` branch selects a fraud model
- Step 4:
  - confirm the downstream dependency path
  - legacy route families call `IMachineLearningEngineeringDataBricksClient`
  - unified/fan-out path calls `IUnifiedMachineLearningClient` and optionally `IMachineLearningClient` for the global return-customer model
  - remember that the executor itself does not currently add retry, timeout budgeting, or max concurrency
- Step 5:
  - inspect route-specific telemetry
  - high-volume production routes currently are:
    - `POST global/return-customer/prediction`
    - `POST predictions`
    - `POST checkout/new-customer/transunion/prediction`
    - `POST checkout/existing-customer/prediction`
    - `POST checkout/new-customer/transunion/v2/prediction`
- Step 6:
  - check the non-request side effects
  - for `POST /machinelearning-gateway/predictions`, confirm whether the request failed before aggregation or succeeded but lost its audit write
- Step 7:
  - capture hand-off evidence
  - include exact endpoint, environment, service entity id, request or failure counts, response-body status and attempt counts, response-time observation, and the specific handler or client anchor you used

# Escalation And Hand-Off

- When to escalate:
  - when failures are no longer isolated to `POST /machinelearning-gateway/predictions`
  - when response times drift materially above the `300ms` default threshold or breach rollout guardrails
  - when Databricks or Event Hubs ownership is clearly outside the gateway team boundary
- Which evidence to carry forward:
  - exact endpoint name
  - exact Dynatrace entity id
  - exact time window
  - request and failure counts
  - response-body status and attempt counts when the route is `POST /machinelearning-gateway/predictions`
  - log or handler evidence showing whether the issue is auth, gateway logic, unified-path request construction, Databricks latency, or Event Hub audit write failure
- Which related teams or services may need to be involved:
  - Decision Engine and Front End Orchestration callers
  - Databricks/model owners
  - Event Hubs or audit-log consumers
  - platform owners if the issue is cluster, pod, or rollout related

# Caveats

- What this guide can answer directly:
  - gateway route inventory
  - production vs development Dynatrace mapping
  - main dependency direction
  - current top-volume and failing production paths
- What requires deeper investigation:
  - model-quality issues
  - downstream Databricks implementation defects
  - caller-side payload construction bugs
- Known telemetry gaps:
  - Smartscape did not expose Databricks as a first-class downstream service in this pass
  - Event Hub audit failures may appear only in logs, not in HTTP failure metrics
  - `RunPredictions` partial model failures can appear as `200 OK`, so HTTP failure metrics alone undercount user-visible degradation

# Related Docs

- Overview page: `./service-overview.md`
- Reference page: `./service-reference.md`
- Tutorial:
  - none yet
