---
title: "API Gateway Pattern"
category: backend-architecture
summary: "A single entry point for all client requests that handles cross-cutting concerns — routing, authentication, rate limiting, SSL termination, and response aggregation — before forwarding to downstream services."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# API Gateway Pattern

> A single entry point for all client requests that handles cross-cutting concerns — routing, authentication, rate limiting, SSL termination, and response aggregation — before forwarding to downstream services.

## Overview

In a microservices architecture, clients would otherwise need to know about every downstream service, handle auth independently, and make dozens of requests to render a single page. The **API Gateway** acts as the system's front door — a reverse proxy that encapsulates the internal service topology, enforces cross-cutting policies, and provides a unified interface to clients.

Major implementations: **Kong**, **AWS API Gateway**, **Envoy**, **Nginx**, **Traefik**, **Apigee**, **Azure API Management**.

## Core Responsibilities

### 1. Request Routing
Map external routes to internal services:

```yaml
# Kong declarative configuration
services:
  - name: order-service
    url: http://order-service:8080
    routes:
      - paths: ["/api/v1/orders"]
        methods: [GET, POST, PUT]

  - name: inventory-service
    url: http://inventory-service:8081
    routes:
      - paths: ["/api/v1/inventory"]
        methods: [GET]
```

### 2. Authentication & Authorization
Centralize auth so individual services don't each implement it:

```yaml
# Kong JWT plugin — validate tokens at gateway
plugins:
  - name: jwt
    config:
      claims_to_verify: [exp]
      key_claim_name: kid
      secret_is_base64: false
```

Services receive pre-validated identity headers (`X-User-ID`, `X-User-Role`) — no auth logic needed internally.

### 3. Rate Limiting
Protect services from overload and enforce fair usage:

```yaml
# Kong rate limiting plugin
plugins:
  - name: rate-limiting
    config:
      minute: 100      # 100 requests/minute per consumer
      hour: 5000
      policy: redis    # distributed counter using Redis
      fault_tolerant: true  # allow traffic if Redis is down
```

### 4. SSL/TLS Termination
Encrypt external traffic; internal service-to-service traffic can use HTTP (or mTLS via service mesh):

```
Client ──HTTPS──► [API Gateway: TLS termination] ──HTTP──► Services
                   (certificate management centralized)
```

### 5. Request/Response Transformation
Adapt between external API contracts and internal service formats:

```lua
-- Kong plugin (Lua): add correlation ID to every request
local function add_correlation_id(conf)
    local correlation_id = kong.request.get_header("X-Correlation-ID")
    if not correlation_id then
        correlation_id = require("resty.random").token(16)
        kong.service.request.set_header("X-Correlation-ID", correlation_id)
    end
end
```

### 6. API Aggregation (Backend for Frontend pattern)
Compose responses from multiple services into one response:

```python
# Gateway-level aggregation (or dedicate to BFF layer)
@app.get("/api/v1/dashboard")
async def dashboard(user_id: str):
    async with asyncio.TaskGroup() as tg:
        orders_task = tg.create_task(order_service.recent(user_id))
        profile_task = tg.create_task(user_service.profile(user_id))
        recs_task = tg.create_task(rec_service.get(user_id))

    return {
        "orders": orders_task.result(),
        "profile": profile_task.result(),
        "recommendations": recs_task.result()
    }
```

## Gateway Architectures

### Single Gateway
One gateway for all clients and all services:
```
Mobile App ──►
Web App    ──► [API Gateway] ──► [Services]
Partner API ──►
```
Simple, but can become a bottleneck and a team coordination chokepoint.

### Multiple Gateways (Per-Client BFF)
Separate gateways optimized per client type:
```
Mobile App ──► [Mobile BFF Gateway] ──► [Services]
Web App    ──► [Web BFF Gateway]    ──► [Services]
Partner    ──► [Partner Gateway]    ──► [Services]
```
Netflix uses this model — mobile gateway returns compressed, mobile-optimized payloads; web gateway returns richer responses.

### Two-Tier Gateway
External-facing public gateway + internal service mesh gateway:
```
Internet ──► [External Gateway: auth, rate limiting, WAF]
               ──► [Internal Gateway: service discovery, routing]
                     ──► [Services]
```

## Major Implementations Compared

| Gateway | Best For | Strengths | Weaknesses |
|---------|----------|-----------|------------|
| **Kong** | Enterprise, highly extensible | Plugin ecosystem, Lua/Go/WASM plugins | Complex config management |
| **AWS API Gateway** | AWS-native serverless | Fully managed, Lambda integration | Vendor lock-in, expensive at high volume |
| **Envoy** | Service mesh, Kubernetes | High performance, xDS protocol | Low-level, complex configuration |
| **Nginx** | High-performance routing | Battle-tested, very fast | Manual plugin development |
| **Traefik** | Kubernetes-native, dynamic | Auto service discovery, Let's Encrypt | Fewer enterprise features |
| **Apigee** | Enterprise API programs | Analytics, developer portal | Very expensive |

## Envoy as API Gateway

```yaml
# Envoy xDS configuration snippet
static_resources:
  listeners:
  - name: listener_0
    address:
      socket_address: { address: 0.0.0.0, port_value: 8080 }
    filter_chains:
    - filters:
      - name: envoy.filters.network.http_connection_manager
        typed_config:
          route_config:
            virtual_hosts:
            - name: backend
              domains: ["*"]
              routes:
              - match: { prefix: "/orders" }
                route: { cluster: order_service }
              - match: { prefix: "/users" }
                route: { cluster: user_service }
```

## Real-World Examples

**Netflix (Zuul / Zuul 2)**: Netflix built Zuul as their API gateway, open-sourced in 2013. Handles ~100,000 requests/second for 200+ services. Zuul 2 moved to async non-blocking I/O. Features: A/B testing via request routing, canary deployments, dynamic routing configuration.

**Uber (Flipr + M3 Gateway)**: Uber's gateway handles routing for their driver and rider apps to 2,000+ microservices, with per-endpoint rate limiting and authentication.

**Airbnb**: Uses Nginx + custom middleware as their API gateway. Centralized authentication, rate limiting per API key, and request logging feeding into their observability stack.

**Amazon API Gateway**: Processes billions of API calls daily for AWS customers. Integrates with Lambda, IAM auth, Cognito, and WAF.

## Observability at the Gateway

The gateway is the ideal place to collect metrics:

```
Metrics per route:
- Request count (total, by status code)
- Latency (p50, p95, p99)
- Request size, response size
- Upstream response time vs. total latency (gateway overhead)
- Rate limit rejected count

Logs:
- Correlation ID (trace across services)
- User ID (anonymized for privacy compliance)
- Geographic region
```

## Trade-offs

| Pros | Cons |
|------|------|
| Centralized auth, rate limiting, SSL | Single point of failure (must be HA) |
| Simplifies clients — one endpoint | Can become a bottleneck under extreme load |
| Decouples client from internal topology | Gateway logic sprawl — business logic leaks in |
| Unified observability and logging | Additional network hop adds latency |
| Easy canary/blue-green routing | Heavyweight gateways add operational complexity |

## Anti-Patterns

- **Business logic in the gateway**: Auth ✅, routing ✅, user enrichment ❌ (belongs in services)
- **One gateway team as bottleneck**: All service changes require gateway team approval
- **No circuit breaking at gateway**: Slow services should be circuit-broken before they affect gateway threads
- **Skipping correlation IDs**: Makes cross-service debugging extremely difficult

## When to Use

✅ **Use when**: Microservices with multiple client types; need for centralized auth/rate limiting; service discovery abstraction; A/B testing or canary deployments at the routing layer.

❌ **Avoid when**: Monolith with standard web framework (use framework middleware instead); very low-traffic systems where operational overhead is too high.

---
*Related: [[Service Mesh]], [[Backend for Frontend]], [[Bulkhead Pattern]], [[Circuit Breaker Pattern]], [[Serverless Architecture]]*
