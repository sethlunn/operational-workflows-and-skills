# Machine Learning Gateway Reference

- Service or system: `provider-gateways / machine-learning-gateway`
- Repo path: `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning`
- Exact scope: deployable inventory, HTTP surface, `RunPredictions` contract and result semantics, runtime entities, config surfaces, and exact evidence anchors for the gateway as reviewed on `2026-04-08`
- Canonical owner:
  - broader `provider-gateways` service definition
  - Confluence anchor: `https://quadpay.atlassian.net/wiki/spaces/QD/pages/1863974918/Credit+Bureau+Gateways`
- Last reviewed: `2026-04-08`

# Deployables

| Name | Type | Entrypoint | Environment notes | Important links |
| --- | --- | --- | --- | --- |
| `machinelearning-gateway` | ASP.NET web API | `Program.cs` -> `Startup.cs` | Production workload `machinelearning-gateway-primary` runs in `eastus-production/api` with `ingress.enabled: false`; development footprint appears in `eastus-dev/master` and `zip-aks-cluster-dev-02/master`, with development ingress enabled | `services/provider-gateways/azure-pipelines-machinelearning-gateway.yml`, `services/provider-gateways/helm/machinelearning-gateway/` |

# HTTP Surface

Base path from code: `/machinelearning-gateway`

| Verb | Path | Purpose | Auth notes | Implementation anchor |
| --- | --- | --- | --- | --- |
| `POST` | `/machinelearning-gateway/predictions` | Concurrent fan-out across the selected model set, always including Unified NCM V1 and optionally Global Return Customer for specific request shapes | `[Authorize(Policy = "Internal")]` | `MachineLearningEngineering/Gateway/Controllers/MachineLearningController.cs`, `MachineLearningEngineering/Gateway/Commands/RunPredictions.cs` |
| `POST` | `/machinelearning-gateway/checkout/new-customer/transunion/reason-codes` | Legacy checkout TransUnion reason-code lookup | Internal policy | `MachineLearningEngineering/Queries/GetCheckoutReasonCodes.cs` |
| `POST` | `/machinelearning-gateway/checkout/new-customer/transunion/v2/reason-codes` | Challenger checkout TransUnion reason-code lookup | Internal policy | `MachineLearningEngineering/Queries/GetCheckoutReasonCodesV2.cs` |
| `POST` | `/machinelearning-gateway/app/signup/transunion/reason-codes` | Legacy app-signup TransUnion reason-code lookup | Internal policy | `MachineLearningEngineering/Queries/GetAppReasonCodes.cs` |
| `POST` | `/machinelearning-gateway/app/signup/transunion/v2/reason-codes` | Challenger app-signup TransUnion reason-code lookup | Internal policy | `MachineLearningEngineering/Queries/GetAppReasonCodesV2.cs` |
| `POST` | `/machinelearning-gateway/checkout/new-customer/transunion/prediction` | Legacy new-customer checkout prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunCheckoutNewCustomerPrediction.cs` |
| `POST` | `/machinelearning-gateway/checkout/new-customer/transunion/v2/prediction` | Challenger new-customer checkout prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunCheckoutNewCustomerPredictionV2.cs` |
| `POST` | `/machinelearning-gateway/checkout/existing-customer/prediction` | Existing-customer checkout prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunCheckoutExistingCustomerPrediction.cs` |
| `POST` | `/machinelearning-gateway/app/signup/transunion/prediction` | Legacy app-signup TransUnion prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunAppTransUnionModelPrediction.cs` |
| `POST` | `/machinelearning-gateway/app/signup/transunion/v2/prediction` | Challenger app-signup TransUnion prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunAppTransUnionV2ModelPrediction.cs` |
| `POST` | `/machinelearning-gateway/app/signup/experian/prediction` | App-signup Experian prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunAppExperianModelPrediction.cs` |
| `POST` | `/machinelearning-gateway/global/return-customer/prediction` | Global return-customer prediction | Internal policy | `MachineLearningEngineering/Gateway/Commands/RunGlobalReturnCustomerPrediction.cs` |
| `GET` | `/machinelearning-gateway/healthz` | Liveness/health check | no controller auth path; mapped by ASP.NET health checks | `Startup.cs` |

# RunPredictions Contract And Result Semantics

- Request contract:
  - request JSON is effectively `camelCase`
  - `customerId` is required and validated as a non-empty GUID
  - `providerBlobPaths` is optional and is not validator-enforced for required keys
  - `additionalFeatures` is optional and is not validator-enforced for required keys
- Live model-selection rules:
  - app channel + `merchant_id` + case-insensitive `transunion` provider blob selects `UnifiedNCMV1PredictionModel` plus `GlobalReturnCustomerPredictionModel`
  - checkout + existing customer selects `UnifiedNCMV1PredictionModel` plus `GlobalReturnCustomerPredictionModel`
  - checkout + new customer + `merchant_id` + case-insensitive `transunion` provider blob currently still selects `UnifiedNCMV1PredictionModel` only
  - default fallback selects `UnifiedNCMV1PredictionModel` only
  - no live `RunPredictions` branch currently selects a fraud model
- Execution and resilience semantics:
  - `ConcurrentFanoutExecutor` starts all selected model tasks with `Task.WhenAll` and returns attempts in strategy order
  - the executor itself does not add retry, timeout budgeting, or configurable max concurrency
  - `GlobalReturnCustomerPredictionModel` uses the decorated `IMachineLearningClient` path with the shared Databricks resilience pipeline
  - `UnifiedNCMV1PredictionModel` uses `IUnifiedMachineLearningClient` and does not go through the same decorator path
- Response semantics:
  - response DTOs serialize in `snake_case`
  - downstream model failures usually still surface as `200 OK` with top-level `Failed` or `PartiallySucceeded` gateway results
  - reachable live top-level statuses are `Succeeded`, `Failed`, and `PartiallySucceeded`
  - `TimedOut` and `NoModelsAvailable` exist in DTOs and factories but are not normally reachable through the current selection and executor path
  - some unified-path request-build failures can become failed attempts with limited `error_messages` detail

# Async Interfaces

| Direction | Name | Purpose | Producer or consumer | Implementation anchor |
| --- | --- | --- | --- | --- |
| Outbound | `MachineLearningRunPredictionsCompleted` | Publish BI/audit data for the fan-out prediction path | Produced by `RunPredictions.Handler`; written through `IAuditHistoryWriter` to `EventHubDecisionEngine` | `MachineLearningEngineering/Gateway/Commands/RunPredictions.cs`, `EventHubs/EventHubsServiceCollectionExtensions.cs`, `EventHubs/BusinessIntelligenceEvents/MachineLearningRunPredictionsCompleted.cs` |

No inbound queue, topic, or worker consumer was found in this service.

# Data Stores And Schemas

- Data stores used:
  - no direct database or cache client was found in the inspected code
  - provider blob-path strings are carried through requests but are not dereferenced by this service in the inspected code
- Important tables, collections, or documents:
  - none found
- Important schema or contract anchors:
  - legacy Databricks client contract:
    - `MachineLearningEngineering/DataBricks/Clients/IMachineLearningEngineeringDataBricksClient.cs`
  - unified Databricks client contract:
    - `MachineLearningEngineering/DataBricks/Clients/IUnifiedMachineLearningClient.cs`
  - fan-out input contract:
    - `MachineLearningEngineering/Gateway/Commands/RunPredictions.cs`
  - provider blob-path key normalization:
    - `MachineLearningEngineering/Gateway/Factories/UnifiedDatabricksRequestFactory.cs`
  - model-selection rules:
    - `MachineLearningEngineering/Gateway/Commands/RunPredictionsCommandExtensions.cs`
  - result-status aggregation:
    - `MachineLearningEngineering/Gateway/Factories/MachineLearningGatewayResultFactory.cs`
  - prediction-model execution:
    - `MachineLearningEngineering/Gateway/Services/ConcurrentFanoutExecutor.cs`
    - `MachineLearningEngineering/Gateway/Models/UnifiedNCMV1PredictionModel.cs`
    - `MachineLearningEngineering/Gateway/Models/GlobalReturnCustomerPredictionModel.cs`
- Data caveats:
  - `RunPredictions.Command.ProviderBlobPaths` is case-sensitive by construction, but selection logic compensates with case-insensitive key checks
  - the request factory remaps `transunion` to `TransUnionCreditReport` and preserves `SocureReport`
  - duplicate provider blob-path keys that collapse after remapping can fail request construction on the unified path

# Runtime Entities

- Dynatrace or platform entities:
  - production service: `SERVICE-F4F7F2376D687A69`
  - development service: `SERVICE-40FFDB56A531E988`
  - production process group: `PROCESS_GROUP-D5E2EC6E18D78A77`
  - development process group: `PROCESS_GROUP-6CAEBDD789416A22`
  - production workload name: `machinelearning-gateway-primary`
  - production cluster and namespace: `eastus-production` / `api`
  - development clusters and namespace: `eastus-dev` / `master`, `zip-aks-cluster-dev-02` / `master`
- Environment mapping:
  - production Smartscape evidence is attached to `SERVICE-F4F7F2376D687A69`
  - development Smartscape evidence is attached to `SERVICE-40FFDB56A531E988`
- Known naming caveats:
  - service-definition microservice name uses hyphens: `machine-learning-gateway`
  - base path, pipeline name, and Helm service name omit the second hyphen: `machinelearning-gateway`

# Config, Flags, And Ownership

- Key config surfaces:
  - `Jwt`
  - `Mediator`
  - `Optimizely`
  - `Swagger`
  - `Polly:EventHub`
  - `Polly:DataBricks`
  - `MachineLearningEngineering`
  - `DatabricksClient`
  - `ConnectionStrings:EventHubDecisionEngine`
- Feature flags:
  - Optimizely-backed `FeatureFlagManager` using `DecisionEngineSdkKey`
- Resilience caveat:
  - `Polly:DataBricks` clearly applies to the decorated `IMachineLearningClient` path and is not the sole control surface for the unified client path used by `UnifiedNCMV1PredictionModel`
- Dashboards, alerts, or SLO anchors:
  - service-definition SLO block in `services/provider-gateways/service.json`
  - production canary thresholds in `services/provider-gateways/helm/machinelearning-gateway/values/production/values.yaml`
  - Dynatrace production service entity `SERVICE-F4F7F2376D687A69`
- Confluence, repo, or service-definition links:
  - `services/provider-gateways/service.json`
  - `https://quadpay.atlassian.net/wiki/spaces/QD/pages/1863974918/Credit+Bureau+Gateways`

# Exact Evidence Anchors

- Important files inspected:
  - `services/provider-gateways/service.json`
  - `services/provider-gateways/azure-pipelines-machinelearning-gateway.yml`
  - `services/provider-gateways/helm/machinelearning-gateway/values/production/values.yaml`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/Program.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/Startup.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/Authentication/StartupAuthentication.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/Gateway/Controllers/MachineLearningController.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/Gateway/Commands/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/Queries/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/Gateway/Models/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/Gateway/Services/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/Gateway/Factories/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/DataBricks/Clients/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/MachineLearningEngineering/DataBricks/Decorators/*.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/Resilience/PollyPipelineSetup.cs`
  - `services/provider-gateways/src/QuadPay.ProviderGateways.MachineLearning/EventHubs/*.cs`
- Important queries used:

```text
smartscapeNodes "SERVICE" | filter id == toSmartscapeId("SERVICE-F4F7F2376D687A69")
smartscapeNodes "SERVICE" | filter id == toSmartscapeId("SERVICE-40FFDB56A531E988")
smartscapeEdges "*" | filter source_id == toSmartscapeId("SERVICE-F4F7F2376D687A69") or target_id == toSmartscapeId("SERVICE-F4F7F2376D687A69")
smartscapeEdges "*" | filter source_id == toSmartscapeId("SERVICE-40FFDB56A531E988") or target_id == toSmartscapeId("SERVICE-40FFDB56A531E988")
timeseries { request_count = sum(dt.service.request.count, scalar: true), failure_count = sum(dt.service.request.failure_count, scalar: true), avg_response_time = avg(dt.service.request.response_time, scalar: true) }, filter:{dt.entity.service == "SERVICE-F4F7F2376D687A69"}, from: now()-24h
timeseries { request_count = sum(dt.service.request.count, scalar: true) }, by:{endpoint.name}, filter:{dt.entity.service == "SERVICE-F4F7F2376D687A69"}, from: now()-24h | sort request_count desc | limit 5
timeseries { failed_request_count = sum(dt.service.request.count, scalar: true) }, by:{endpoint.name, http.response.status_code}, filter:{dt.entity.service == "SERVICE-F4F7F2376D687A69" and failed == true}, from: now()-24h | sort failed_request_count desc | limit 20
```

- Important external links:
  - `https://iec67099.apps.dynatrace.com`
  - `https://quadpay.atlassian.net/wiki/spaces/QD/pages/1863974918/Credit+Bureau+Gateways`

# Related Docs

- Overview page: `./service-overview.md`
- Operability guide: `./operability-guide.md`
- Tutorial:
  - none yet
