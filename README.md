# specmatic-prototype-contracts: central API contract repository (simulated)

This is the single source of truth for the HTTP contracts between the prototype's services.
It is a separate git repository: services never edit specs in their own repos. They use
the specs from here, pinned by tag.

```
common/
  common.yaml                                shared ProblemDetails schema + 400/401/404/502 responses
specs/
  app-a-production/
    well-registry-service.yaml
    well-registry-service_examples/          consumer-contributed examples (external JSON)
    production-forecast-service.yaml
    production-forecast-service_examples/
  app-b-change-mgmt/
    change-request-service.yaml
    change-request-service_examples/
    approval-service.yaml
    approval-service_examples/
consumers.yaml                               who relies on which endpoint/fields (governance; not read by Specmatic)
```

## Conventions
- OpenAPI 3.0.3. `servers: /v2` documents the base path. Specmatic ignores it, so tests
  pass `--testBaseURL http://<host>:<port>/v2`.
- Health probes (`/liveness`, `/readiness`) are deliberately **not** in the specs.
- Every operation declares 400/401 (and 404 where there is an id). Errors use
  `application/problem+json` (RFC 9457). `errors` is optional.
- **Inline named examples** (provider-authored): a parameter or request example and a
  response example with the **same name** form one request/response pair.
- **External examples** in `<spec-name>_examples/`: each file name starts with the consumer
  that relies on it, e.g. `production-forecast__get_well_W-001_active.json`. The provider
  runs them as contract tests, and the consumer gets them as stub responses.
- Validate before committing:
  ```powershell
  docker run --rm -v "${PWD}:/usr/src/app" -w /usr/src/app specmatic/specmatic:2.55.0 `
    examples validate --spec-file specs/app-a-production/well-registry-service.yaml --examples-to-validate BOTH
  ```

## Change rules
- A spec change goes through a pull request and must pass the backward-compatibility check
  against the previous tag.
- Releases are tagged (`v1`, `v2`, ...). Providers test against a tag; consumers stub from a tag.

License: MIT.
