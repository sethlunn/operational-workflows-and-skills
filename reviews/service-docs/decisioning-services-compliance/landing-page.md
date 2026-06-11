# Compliance

This page is the stable landing page for the `decisioning-services / compliance` documentation set.

Published on `2026-04-21` from repo-managed source.

## Scope

* ASP.NET Compliance API under `services/compliance/src/Zip.Compliance`
* four projection workers under `services/compliance/src/Zip.Compliance.Projections.*`
* production-focused runtime mapping and current telemetry on `2026-04-21`
* code-backed dependency, storage, and contract inventory
* operational health and debugging guidance

## Diataxis Set

Child pages under this page:

* [Compliance - Overview](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395546162/Compliance+-+Overview)
* [Compliance - Reference](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5395185701/Compliance+-+Reference)
* [Compliance - Operability Guide](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5396070408/Compliance+-+Operability+Guide)
* [Compliance - AAN Generation](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5397348353/Compliance+-+AAN+Generation)
* tutorial only when there is a real guided first-run need

## Related Existing Pages

* [Compliance](https://quadpay.atlassian.net/wiki/spaces/QD/pages/4606001162/Compliance)
* [Compliance Endpoint Traffic Analysis](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5266636805/Compliance+Endpoint+Traffic+Analysis)
* [Operation Clarity - Compliance Service Codebase Documentation & Refactor](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5250449424/Operation+Clarity+Compliance+Service+Codebase+Documentation+Refactor)
* [CreateAdverseActionNotice Flow Map](https://quadpay.atlassian.net/wiki/spaces/QD/pages/5250678813/CreateAdverseActionNotice+Flow+Map)

## Notes

* Source of truth should remain in git.
* This page should stay stable and link outward to supporting deep dives rather than trying to absorb all of them.
* Child pages should be updated in place rather than recreated.
* The service spans one HTTP API plus four background projection deployables, so the overview intentionally treats Compliance as a small system rather than a single web controller surface.
