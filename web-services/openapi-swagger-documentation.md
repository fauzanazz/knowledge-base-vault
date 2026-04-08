---
title: "OpenAPI & Swagger Documentation"
category: web-services
summary: "Use OpenAPI 3.1 as a contract for API design, documentation, code generation, and automated validation."
sources:
  - web-research-2026
updated: 2026-04-08
---
# OpenAPI & Swagger Documentation
> Use OpenAPI 3.1 as a contract for API design, documentation, code generation, and automated validation.

## What Is OpenAPI?

OpenAPI (formerly Swagger) is a language-agnostic specification for describing HTTP APIs in YAML or JSON. OpenAPI 3.1 (released 2021) aligns fully with JSON Schema Draft 2020-12, closing the long-standing divergences between OpenAPI schemas and standard JSON Schema.

**Key terms:**
- **OpenAPI Specification (OAS):** The standard itself.
- **Swagger:** Historically the spec name; now refers to the tooling ecosystem (Swagger UI, Swagger Editor, Swagger Codegen).
- **Spec file:** Your `openapi.yaml` — the source of truth.

---

## OAS 3.1 Spec Structure

```yaml
openapi: "3.1.0"
info:
  title: Orders API
  version: "2026-01-01"
  description: |
    Manages customer orders. Breaking changes follow the versioning policy.
  contact:
    email: api-support@example.com

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://sandbox.api.example.com/v1
    description: Sandbox

paths:
  /orders:
    post:
      operationId: createOrder
      summary: Create a new order
      tags: [Orders]
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateOrderRequest'
            examples:
              minimal:
                $ref: '#/components/examples/MinimalOrder'
      responses:
        '201':
          description: Order created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Order'
        '422':
          $ref: '#/components/responses/ValidationError'

components:
  schemas:
    Order:
      type: object
      required: [id, status, amount]
      properties:
        id:
          type: string
          format: uuid
          readOnly: true
        status:
          type: string
          enum: [pending, confirmed, shipped, cancelled]
        amount:
          type: integer
          description: Amount in minor currency units (cents)
          minimum: 1
    CreateOrderRequest:
      type: object
      required: [items]
      properties:
        items:
          type: array
          minItems: 1
          items:
            $ref: '#/components/schemas/OrderItem'
  responses:
    ValidationError:
      description: Request failed validation
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/ProblemDetails'
  securitySchemes:
    BearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - BearerAuth: []
```

**`components/`** is the reuse hub. Define schemas, responses, parameters, examples, and security schemes once; `$ref` everywhere else.

---

## Schema-First vs Code-First

### Schema-First (Design-First)
Write `openapi.yaml` before any code. Generate server stubs and client SDKs from the spec.

**Advantages:**
- Contract is explicit before implementation — forces API design discussion.
- Teams can work in parallel: frontend generates a mock server from the spec while backend implements.
- Breaking changes are visible as diff on the spec file.
- CI can lint and validate the spec independently of running code.

**Disadvantages:**
- Requires discipline to keep spec and implementation in sync.
- Extra build step for code generation.

### Code-First
Annotate code (decorators, comments) and generate the spec from the running application.

```python
# FastAPI example — generates OpenAPI spec automatically
@app.post("/orders", response_model=Order, status_code=201)
def create_order(body: CreateOrderRequest) -> Order:
    ...
```

**Advantages:**
- Spec is always in sync (generated from code).
- Less overhead for fast-moving internal APIs.

**Disadvantages:**
- API shape is driven by implementation details, not consumer needs.
- Harder to design good APIs when you're thinking in framework terms.

**Recommendation for external APIs:** Schema-first. Use [Spectral](https://stoplight.io/open-source/spectral) to lint the spec in CI, and contract tests to verify implementation matches.

---

## Code Generation

### Client SDKs
```bash
# OpenAPI Generator (openapi-generator.tech)
openapi-generator-cli generate \
  -i openapi.yaml \
  -g typescript-fetch \
  -o ./clients/typescript

# Also supports: python, java, go, ruby, swift, kotlin, ...
```

Generated clients include: typed models, request/response marshaling, auth header injection, retry logic hooks. Regenerate on every spec change.

### Server Stubs
```bash
openapi-generator-cli generate \
  -i openapi.yaml \
  -g python-fastapi \
  -o ./server-stub
```

Stubs generate route definitions and model classes. Implement business logic in the generated handler functions; don't edit generated files directly (use extension points).

### Alternatives
- **oapi-codegen** (Go) — idiomatic Go, widely used.
- **Kiota** (Microsoft) — generates SDK for any language with a plugin model.
- **Speakeasy** — commercial, produces high-quality SDKs with better ergonomics than OSS generators.

---

## Interactive Documentation: Swagger UI & Redoc

### Swagger UI
The classic "try it out" interface. Renders the spec as an interactive form — fill fields, click Execute, see real HTTP requests.

```html
<!-- Self-hosted in 3 lines -->
<link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist/swagger-ui.css">
<div id="swagger-ui"></div>
<script src="https://unpkg.com/swagger-ui-dist/swagger-ui-bundle.js"></script>
<script>
  SwaggerUIBundle({ url: "/openapi.yaml", dom_id: '#swagger-ui' });
</script>
```

### Redoc
Better for documentation-heavy APIs. Three-panel layout: navigation, description, code samples. Read-only (no "try it out"). Cleaner rendering for complex nested schemas.

```bash
npx @redocly/cli build-docs openapi.yaml -o docs/index.html
```

### Recommendation
Host both: Redoc as the primary developer portal, Swagger UI as a testing sandbox. Gate Swagger UI behind auth in production environments.

---

## Request/Response Validation

Validate incoming requests against the spec using middleware — catch malformed payloads early, before business logic.

**Python (openapi-core):**
```python
from openapi_core import OpenAPI
spec = OpenAPI.from_file_path("openapi.yaml")

@app.middleware("http")
async def validate_request(request, call_next):
    try:
        spec.validate_request(request)
    except ValidationError as e:
        return JSONResponse(status_code=422, content={"errors": str(e)})
    return await call_next(request)
```

**Node.js (express-openapi-validator):**
```js
app.use(OpenApiValidator.middleware({
  apiSpec: './openapi.yaml',
  validateRequests: true,
  validateResponses: true  // Catches implementation bugs in dev
}));
```

Enable **response validation in development and CI** — it surfaces implementation bugs where your code returns a shape that doesn't match the spec. Disable in production (performance overhead, may break clients on schema changes).

---

## Versioning OpenAPI Specs

**The spec version ≠ the API version.** Manage both separately.

```yaml
info:
  version: "2026-01-01"  # API version (date-based or semver)
```

**Strategies:**
1. **One file per major version:** `openapi-v1.yaml`, `openapi-v2.yaml`. Simple, full independence.
2. **Overlay/extension pattern:** Maintain a base spec plus diff patches per version. Complex but DRY.
3. **Git tags:** Tag the spec repo at each published API version. Use `git diff v1 v2 -- openapi.yaml` to generate changelogs.

**Breaking vs non-breaking changes:**
- **Breaking:** Removing a field, changing a type, making an optional field required, renaming an `operationId`.
- **Non-breaking:** Adding an optional field, adding a new endpoint, relaxing a validation constraint.

Use [openapi-diff](https://github.com/OpenAPITools/openapi-diff) in CI to automatically classify changes and block breaking changes on stable API versions.

---

## Contract Testing

Contract tests verify that the implementation matches the spec — catching drift before it reaches production.

**Schemathesis (property-based testing):**
```bash
# Generates and runs hundreds of test cases from your spec
schemathesis run openapi.yaml --url http://localhost:8000 \
  --checks all \
  --stateful=links
```

**Dredd:**
```bash
dredd openapi.yaml http://localhost:8000
```

**In CI pipeline:**
```yaml
- name: Contract tests
  run: |
    docker-compose up -d app
    schemathesis run openapi.yaml --url http://localhost:8000 \
      --checks not_a_server_error,status_code_conformance,response_schema_conformance
```

Contract tests are most valuable when you own both the spec and the implementation — they're the automated guard against "we updated the spec but forgot to update the handler."

---

## Best Practices

**Always include:**
- `operationId` on every operation (used for code gen method names).
- `description` on every property and operation (appears in generated docs and SDKs).
- At least one `example` per schema (generated clients use these as test fixtures).
- A reusable error schema (`ProblemDetails` per RFC 7807 / RFC 9457).

**Error schema (RFC 9457):**
```yaml
ProblemDetails:
  type: object
  required: [type, title, status]
  properties:
    type:
      type: string
      format: uri
      example: "https://api.example.com/errors/validation-failed"
    title:
      type: string
    status:
      type: integer
    detail:
      type: string
    errors:
      type: array
      items:
        type: object
```

**Avoid:**
- `additionalProperties: false` on response schemas (breaks forward compatibility).
- Deeply nested `allOf`/`oneOf` chains — hard to read and generates ugly code.
- Using `type: object` without defining `properties` (gives up schema validation).
- Overly generic error responses (`"message": "string"`) with no machine-readable type.

---

## Trade-offs Summary

| Decision | Schema-First | Code-First |
|----------|-------------|------------|
| API design quality | Higher | Lower |
| Implementation sync risk | Higher | Lower |
| Onboarding speed | Slower | Faster |
| Best for | External/public APIs | Internal/fast-moving APIs |

| Tool | Use Case |
|------|---------|
| Swagger UI | Interactive testing sandbox |
| Redoc | Developer-facing documentation portal |
| Spectral | Spec linting in CI |
| Schemathesis | Contract/fuzz testing |
| OpenAPI Generator | Client SDK + server stub generation |
| openapi-diff | Detect breaking changes in CI |
