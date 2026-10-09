---
generated: '2026-10-09'
method: generated
name: lint-an-openapi-spec
description: Check an OpenAPI document against Schwarz company API rules using a self-hosted Schwarz IT API Linting Service.
api: openapi/schwarz-it-api-linter-service-openapi.yml
operations: [HealthProbeController_returnLive, RulesController_getCompanyApiRules, LintingsController_createLinting]
source: >-
  Grounded in openapi/schwarz-it-api-linter-service-openapi.yml (from
  https://raw.githubusercontent.com/SchwarzIT/api-linter-service/main/openapi/openapi.json). Every
  operationId verified verbatim in that spec.
---

# Lint an OpenAPI spec against Schwarz company API rules

The service is self-hosted: there is no Schwarz-operated public base URL. Use your own deployment (the contract's default is `http://localhost:3001`). The contract declares no authentication.

## Steps

1. **Check the service is up** — `HealthProbeController_returnLive` (`GET /.well-known/live`). Expect 200.
2. **(Optional) Read the ruleset** — `RulesController_getCompanyApiRules` (`GET /api-linting/api/v1/rules?apiType=...`), where `apiType` is one of `backend_for_frontend`, `legacy_api`, `product_api`. Returns the Spectral rules as YAML. 400 on a missing/invalid `apiType`.
3. **Lint the spec** — `LintingsController_createLinting` (`POST /api-linting/api/v1/lintings`) with JSON body `{"apiType": "product_api", "apiSpecAsBase64": "<base64 of your spec>"}`.
   - 201 returns `CreatedLintingDto`: a `description` saying whether the spec complies, `linkApiRules`, and the Spectral results (`SpectralResult` with code, message, path, severity, range).
   - 400 means mandatory body properties are missing — fix the request, do not retry unchanged.

## Rules

- Base64-encode the whole spec file; do not send raw YAML/JSON in `apiSpecAsBase64`.
- Use the same `apiType` in steps 2 and 3 so the rules you read are the rules applied.
- Linting has no documented persistent side effect, so retrying a failed network call is safe.
