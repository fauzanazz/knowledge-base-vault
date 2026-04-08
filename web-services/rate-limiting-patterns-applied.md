---
title: "Rate Limiting Patterns: Applied"
category: web-services
summary: "Production-ready rate limiting: distributed implementations, tiered limits, headers, quota management, and graceful degradation."
sources:
  - web-research-2026
updated: 2026-04-08
---
# Rate Limiting Patterns: Applied
> Production-ready rate limiting: distributed implementations, tiered limits, headers, quota management, and graceful degradation.

This article assumes familiarity with basic algorithms (token bucket, leaky bucket, fixed window). It focuses on **applied engineering**: how to implement these patterns at scale in real distributed systems.

---

## Sliding Window Log vs. Sliding Window Counter (Distributed)

### Sliding Window Log
Stores every request timestamp in a sorted set (e.g., Redis `ZSET`). On each request:
1. Remove all entries older than `now - window`.
2. Count remaining entries.
3. If count < limit, add current timestamp and allow.

```lua
-- Redis Lua script: sliding window log
local key = KEYS[1]
local now = tonumber(ARGV[1])
local window = tonumber(ARGV[2])
local limit = tonumber(ARGV[3])

redis.call('ZREMRANGEBYSCORE', key, 0, now - window)
local count = redis.call('ZCARD', key)
if count < limit then
  redis.call('ZADD', key, now, now)
  redis.call('EXPIRE', key, window / 1000)
  return 1  -- allowed
end
return 0  -- denied
```

**Trade-offs:** Exact accuracy, but memory grows with request volume (O(requests) per key). Impractical for high-traffic keys.

### Sliding Window Counter
Hybrid: combines two fixed-window counters (current + previous) weighted by overlap position.

```
rate ≈ prev_count × ((window - elapsed) / window) + curr_count
```

- **Memory:** O(1) per key — only two counters + timestamps.
- **Accuracy:** ~99%+ under uniform traffic; slight over/under-counting at window boundaries.
- **Preferred in production** due to low memory footprint.

---

## Redis-Based Distributed Rate Limiting

### MULTI/EXEC (Optimistic)
```python
pipe = redis.pipeline()
pipe.multi()
pipe.incr(key)
pipe.expire(key, window_seconds)
results = pipe.execute()
count = results[0]
```
**Problem:** Race between INCR and EXPIRE if key doesn't exist yet. Use `SET key 0 EX window NX` to initialize atomically, then INCR.

### Lua Scripts (Preferred)
Lua executes atomically in Redis — no race conditions, no network round-trips between steps.

```lua
-- Sliding window counter (atomic)
local key = KEYS[1]
local prev_key = KEYS[2]
local limit = tonumber(ARGV[1])
local now = tonumber(ARGV[2])
local window_ms = tonumber(ARGV[3])

local prev = tonumber(redis.call('GET', prev_key) or 0)
local curr = tonumber(redis.call('GET', key) or 0)
local elapsed = now % window_ms
local weighted = math.floor(prev * (window_ms - elapsed) / window_ms) + curr

if weighted >= limit then return 0 end

redis.call('INCR', key)
redis.call('PEXPIRE', key, window_ms * 2)
return 1
```

**Deployment:** Use `EVALSHA` with cached script SHA for efficiency. Handle `NOSCRIPT` error by re-loading with `EVAL`.

### Redis Cluster Considerations
- Keys for the same user must hash to the same slot. Use hash tags: `{user:123}:ratelimit`.
- Avoid cross-slot Lua scripts — they'll error in cluster mode.
- For global rate limits, use a dedicated single-shard Redis or a central coordination service.

---

## Tiered Rate Limiting

Layer multiple limits, evaluated in order (most specific first):

| Tier       | Key                        | Limit Example     | Purpose                          |
|------------|----------------------------|-------------------|----------------------------------|
| Per-user   | `rl:user:{user_id}`        | 1000 req/min      | Fair use per account             |
| Per-API-key| `rl:key:{api_key}`         | 500 req/min       | Programmatic client limits       |
| Per-IP     | `rl:ip:{ip}`               | 100 req/min       | Unauthenticated / abuse defense  |
| Global     | `rl:global:{endpoint}`     | 50,000 req/min    | System-wide capacity cap         |

**Evaluation strategy:** Check tiers fastest-to-most-expensive. Deny on first breach. Include which tier triggered the 429 in the response (`X-RateLimit-Policy: per-ip`).

**Pitfall:** Don't skip per-IP limits for authenticated users — compromised accounts still need IP-based defense.

---

## Rate Limit Headers

Standard headers (IETF draft + de-facto conventions):

```http
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 743
X-RateLimit-Reset: 1712611200        # Unix timestamp of window reset
Retry-After: 47                      # Seconds until retry (on 429)
X-RateLimit-Policy: per-user         # Which tier triggered (non-standard)
```

**On 429 responses:**
```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 23
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1712611200

{"error": "rate_limit_exceeded", "retry_after": 23}
```

**IETF RateLimit header (RFC draft):** Newer spec combines into one header:
```
RateLimit: limit=1000, remaining=743, reset=47
```
Not yet universally supported — emit both forms during transition.

---

## Quota Management: Burst vs. Sustained vs. Credit-Based

### Burst + Sustained (Two-Layer Token Bucket)
```
Burst:     100 requests in any 1-second window
Sustained: 1000 requests per minute

Check both; deny if either is exceeded.
```

Useful for allowing short legitimate spikes (paginating results) without enabling sustained abuse.

### Credit-Based (Cost-per-Endpoint)
Assign costs to endpoints based on resource consumption:

| Endpoint              | Credit Cost | Rationale                         |
|-----------------------|-------------|-----------------------------------|
| `GET /users/me`       | 1           | Cheap read                        |
| `GET /search?q=...`   | 5           | Full-text index hit               |
| `POST /export/csv`    | 20          | DB scan + file generation         |
| `POST /ai/generate`   | 50          | GPU/LLM inference                 |

User bucket: 1000 credits/hour. A single `/ai/generate` call costs 50; a user can make 20 before hitting their quota, regardless of total request count.

**Implementation:** Same sliding-window or token-bucket, but `INCRBY cost` instead of `INCR 1`.

---

## API Gateway Rate Limiting

### Kong
```yaml
plugins:
  - name: rate-limiting
    config:
      minute: 1000
      hour: 10000
      policy: redis          # local | cluster | redis
      redis_host: redis
      limit_by: consumer     # ip | credential | consumer | header | path
      hide_client_headers: false
```

### Envoy (via `local_ratelimit` or external `ratelimit` service)
```yaml
http_filters:
  - name: envoy.filters.http.local_ratelimit
    typed_config:
      token_bucket:
        max_tokens: 1000
        tokens_per_fill: 1000
        fill_interval: 60s
      filter_enabled: { default_value: { numerator: 100 } }
```
For distributed limits, Envoy integrates with Lyft's `ratelimit` gRPC service backed by Redis.

### nginx (`limit_req`)
```nginx
limit_req_zone $binary_remote_addr zone=per_ip:10m rate=100r/m;
limit_req_zone $http_x_api_key      zone=per_key:10m rate=500r/m;

location /api/ {
    limit_req zone=per_ip  burst=20 nodelay;
    limit_req zone=per_key burst=50 nodelay;
    limit_req_status 429;
}
```
**Note:** nginx uses leaky bucket with `burst` queue. `nodelay` processes burst immediately (no queuing delay) but still counts toward the burst budget.

---

## Graceful Degradation

### 429 Response Best Practices
- Always include `Retry-After` — clients that don't get it will hammer you.
- Return machine-readable error codes: `{"code": "RATE_LIMITED", "tier": "per-user"}`.
- Log rate limit events with user/IP for abuse detection, not just metrics.

### Queue Overflow + Backpressure
When 429 is too blunt, queue excess requests:
1. Accept the request, return `202 Accepted` with a polling URL.
2. Worker drains the queue at the rate limit pace.
3. Client polls or uses webhook callback.

Use for expensive async operations (exports, bulk imports). Not appropriate for real-time APIs.

### Backpressure in Service Mesh
Envoy circuit breakers + rate limits together:
- Rate limit: protects downstream (DB, external APIs).
- Circuit breaker: protects the service itself from overload.
- Use both: rate limit at ingress, circuit breaker at egress.

---

## Common Pitfalls

| Pitfall | Fix |
|---------|-----|
| Rate limiting in app code without distributed state | Move to Redis or API gateway |
| Fixed window with boundary burst (2× spike) | Use sliding window counter |
| No `Retry-After` header on 429 | Always include it |
| Global limit only (no per-user) | One noisy client starves all users |
| Forgetting IPv6 — `/128` vs `/64` for IP limits | Rate limit on `/64` subnet prefix |
| Rate limiting health check endpoints | Whitelist `/health`, `/metrics` |
| Not accounting for clock skew in distributed counters | Use Redis server time (`TIME` command) |
