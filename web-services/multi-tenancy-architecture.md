---
title: "Multi-Tenancy Architecture"
category: web-services
summary: "Isolation models, row-level security, noisy neighbor mitigation, tenant-aware routing, compliance requirements, and migration strategies for multi-tenant SaaS systems."
sources:
  - web-research-2026
updated: 2026-04-08
---
# Multi-Tenancy Architecture
> Isolation models, row-level security, noisy neighbor mitigation, tenant-aware routing, compliance requirements, and migration strategies for multi-tenant SaaS systems.

## Isolation Models

The fundamental trade-off in multi-tenancy is **isolation vs. efficiency**. More isolation means more operational overhead, more cost, but stronger guarantees.

### Model 1: Shared Database, Shared Schema (Row-Level Isolation)
All tenants share one database and one set of tables. Every table has a `tenant_id` column.

```sql
CREATE TABLE orders (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   UUID NOT NULL REFERENCES tenants(id),
  customer_id UUID NOT NULL,
  total       NUMERIC(12,2),
  created_at  TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_orders_tenant ON orders(tenant_id);
```

**Pros:** Simplest to operate. One schema to migrate. Efficient resource utilization. Works for hundreds of thousands of tenants.
**Cons:** A bug can leak cross-tenant data. Harder to comply with data residency requirements. One large tenant can affect query performance for others (noisy neighbor). Backup/restore is all-or-nothing.

**Best for:** High-volume, small-tenant SaaS (e.g., project management, CRM for SMBs).

### Model 2: Shared Database, Schema-per-Tenant
One database instance, but each tenant gets their own schema (PostgreSQL `search_path`).

```sql
-- Provisioning a tenant
CREATE SCHEMA tenant_acme;
CREATE TABLE tenant_acme.orders ( ... );  -- schema-level isolation
SET search_path = tenant_acme;            -- per-connection routing
```

**Pros:** Hard isolation boundary at the schema level. Tenant migrations can be independent. Easy per-tenant backup (pg_dump a single schema). Simpler cross-tenant queries for analytics (when needed).
**Cons:** Schema count creates operational complexity at scale (1000+ schemas = 1000+ migration targets). Connection pooling must be schema-aware (PgBouncer doesn't handle this well natively).

**Best for:** Mid-market SaaS, 10–10,000 tenants, where compliance or data isolation matters but full DB-per-tenant is too costly.

### Model 3: Database-per-Tenant (Silo Model)
Each tenant has a completely separate database instance (or cluster).

**Pros:** Maximum isolation — a tenant's outage, breach, or noisy workload does not affect others. Trivial compliance (tenant data is physically isolated). Independent backup, restore, and point-in-time recovery.
**Cons:** Massive operational overhead. Running 10,000 DB instances is expensive and complex. Schema migrations must be applied to every DB (tooling like Flyway multi-tenant or custom orchestration required). Connection pool explosion.

**Best for:** Enterprise/large-customer tiers, regulated industries (healthcare, finance), data residency requirements, or tenants who contractually require isolation.

---

## Row-Level Security (RLS)

PostgreSQL's Row-Level Security enforces tenant isolation *at the database level*, making it impossible to query another tenant's data even with a misconfigured application.

```sql
-- Enable RLS on the table
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

-- Policy: users can only see their tenant's rows
CREATE POLICY tenant_isolation ON orders
  USING (tenant_id = current_setting('app.tenant_id')::UUID);

-- Application sets the tenant context at connection time
SET app.tenant_id = '550e8400-e29b-41d4-a716-446655440000';

-- Now this query is automatically filtered:
SELECT * FROM orders;  -- implicitly adds WHERE tenant_id = '550e...'
```

### RLS with PgBouncer / Connection Pooling
Transaction-mode poolers (PgBouncer in transaction mode) reset session variables between transactions. Use `SET LOCAL` inside transactions, or use `SET` only in session mode with dedicated connections per tenant.

```python
# Application-level enforcement (belt-and-suspenders with RLS)
class TenantScopedSession:
    def __init__(self, tenant_id: str):
        self.tenant_id = tenant_id

    def query(self, model):
        return db.session.query(model).filter_by(tenant_id=self.tenant_id)
```

**Important:** Never build queries where tenant_id is injected from user input without validation. Validate tenant_id against the authenticated JWT/session before setting the DB context.

---

## Compute Isolation

### Shared Pool (Default)
All tenants run on the same application servers. Easiest to operate, lowest cost.

**Risk:** CPU/memory spikes from one tenant slow down others. A tenant running a bulk export can cause p99 latency spikes for all tenants.

### Dedicated Pool (Tenant Pinning)
Route specific tenants (usually large/enterprise ones) to dedicated compute pools. Can be implemented via tenant-aware load balancer routing.

```yaml
# Kong / Nginx upstream routing example
- tenant: "enterprise-acme"
  upstream: "pool-dedicated-acme"
  resources: { cpu: 8, memory: "32Gi" }
- tenant: "*"
  upstream: "pool-shared"
  resources: { cpu: 2, memory: "8Gi" }
```

### Resource Quotas
Enforce per-tenant rate limits and resource caps at the application layer or API gateway.

```python
# Redis-backed token bucket per tenant
def check_rate_limit(tenant_id: str, limit: int, window_seconds: int) -> bool:
    key = f"ratelimit:{tenant_id}"
    pipe = redis.pipeline()
    pipe.incr(key)
    pipe.expire(key, window_seconds)
    count, _ = pipe.execute()
    return count <= limit
```

Use separate rate limit tiers per pricing plan: free tier gets 100 req/min, pro gets 1,000, enterprise gets custom.

---

## Noisy Neighbor Problem

### Detection
Track per-tenant resource consumption metrics:
- DB query time per tenant (tag Prometheus metrics with `tenant_id`)
- CPU time per tenant (use cgroups or container-level metrics)
- Queue depth per tenant (if using task queues)

```sql
-- PostgreSQL: identify heavy tenant queries
SELECT
  (regexp_match(query, 'tenant_id\s*=\s*''([^'']+)'''))[1] AS tenant,
  calls,
  mean_exec_time,
  total_exec_time
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 20;
```

### Mitigation Strategies
1. **Query timeouts per tenant:** Set `statement_timeout` when executing tenant queries.
2. **Async job queuing with per-tenant fairness:** Use fair-queue schedulers (e.g., Celery with priority queues or Redis Streams with consumer groups per tenant).
3. **Request shedding:** During high load, preferentially shed requests from free-tier tenants.
4. **Tenant offloading:** Automatically migrate a noisy tenant to their own infrastructure tier.

---

## Tenant-Aware Routing

### Subdomain-Based Routing
Each tenant gets a subdomain. Common for white-label products.

```
acme.myapp.com     → tenant: acme
globocorp.myapp.com → tenant: globocorp
```

**Resolution:** Wildcard DNS (`*.myapp.com → LB`). Load balancer or application extracts subdomain from `Host` header to identify tenant.

```python
def get_tenant_from_request(request) -> str:
    host = request.headers.get("Host", "")
    subdomain = host.split(".")[0]
    tenant = TenantRegistry.lookup(subdomain)
    if not tenant:
        raise TenantNotFoundError(subdomain)
    return tenant
```

### Header-Based Routing
Clients send a tenant identifier in a header. Common for API-first products.

```http
GET /api/orders HTTP/1.1
X-Tenant-ID: acme-corp
Authorization: Bearer eyJ...
```

**Security:** Always validate that the authenticated user belongs to the claimed tenant. Never trust `X-Tenant-ID` without cross-checking against the JWT's tenant claim.

### Path-Based Routing (for internal/B2B)
```
/tenants/{tenant_slug}/api/orders
```
Rarely used for end-user products; common for admin dashboards or multi-tenant CLIs.

---

## Data Residency and Compliance

### GDPR Per-Tenant
Each EU tenant's personal data must remain in the EU. Implementation options:

1. **Region-tagged tenants:** Assign each tenant to a region at creation. Route all requests and store all data in that region.
   ```json
   { "tenant_id": "acme", "data_region": "eu-west-1", "gdpr_subject": true }
   ```

2. **Federated databases:** Run separate DB clusters per region. Tenant metadata (non-PII) may live in a global registry; PII data lives only in the tenant's region DB.

3. **Per-tenant encryption keys (BYOK):** Store tenant data encrypted with a per-tenant key. For GDPR "right to erasure," delete the key instead of purging individual rows — crypto-shredding.
   ```python
   # Crypto-shredding: delete tenant key = data becomes unreadable
   def erase_tenant_data(tenant_id: str):
       kms.schedule_key_deletion(
           KeyId=f"alias/tenant-{tenant_id}",
           PendingWindowInDays=7
       )
       # Data in DB is now permanently unreadable
   ```

### Audit Logs
Maintain per-tenant, immutable audit logs. Tenants may need to export their audit trail. Use write-once S3 bucket policies or append-only log tables.

---

## Migration Between Tiers and Tenant Onboarding

### Onboarding Pipeline
```
1. Create tenant record (tenant_id, slug, plan, region, created_at)
2. Provision data store (create schema or DB depending on tier)
3. Apply migrations to new tenant schema
4. Seed default data (roles, settings templates)
5. Create initial admin user account
6. Configure routing (add subdomain DNS, register in LB)
7. Send welcome email with setup instructions
```
This entire flow should be automated and idempotent. Use saga/workflow patterns (Temporal, AWS Step Functions) for reliable multi-step provisioning.

### Migrating Between Isolation Tiers (Shared → Dedicated)
The hardest migration: moving a tenant from shared schema to their own database when they upgrade to enterprise.

```
Strategy: Export → Provision → Sync → Cut over → Verify → Clean up

1. Export tenant data: pg_dump --schema=tenant_acme or filtered dump
2. Provision new dedicated DB with schema migrations applied
3. Import data snapshot
4. Enable CDC (Change Data Capture) to sync deltas during cutover window
5. Pause writes for tenant (maintenance mode, seconds to minutes)
6. Apply final delta; verify row counts match
7. Update routing to point to new DB
8. Resume writes; verify
9. Remove data from shared DB after retention window
```

**Key concern:** Zero downtime migrations require CDC tooling (Debezium, pglogical). For most SaaS scenarios, a brief maintenance window (< 5 min) is acceptable and vastly simpler.

### Tenant Offboarding
- Export all data in a portable format (CSV, JSON) — required by GDPR Art. 20
- Provide 30–90 day data retention after contract end
- Securely delete data (or crypto-shred) with confirmation
- Revoke all API keys and sessions immediately upon cancellation
