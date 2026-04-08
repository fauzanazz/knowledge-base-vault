---
title: "Webhook Design Patterns"
category: web-services
summary: "Design robust webhook systems with secure delivery, idempotent consumers, and reliable failure handling."
sources:
  - web-research-2026
updated: 2026-04-08
---
# Webhook Design Patterns
> Design robust webhook systems with secure delivery, idempotent consumers, and reliable failure handling.

## Push vs Pull: Why Webhooks?

**Pull (polling):** Consumers periodically hit an endpoint to check for new events. Simple to implement but wasteful — most polls return nothing, and latency equals poll interval.

**Push (webhooks):** The producer POSTs events to a consumer-registered URL when something happens. Near-real-time, efficient, but shifts reliability burden to the producer.

| Dimension        | Polling            | Webhooks                    |
|-----------------|--------------------|-----------------------------|
| Latency          | Poll interval (s–m) | Near real-time (<1s)        |
| Infrastructure   | Consumer bears load | Producer bears load         |
| Reliability      | Consumer controls   | Producer must retry         |
| Firewall-friendly | Yes               | Needs inbound port open     |
| Debugging        | Easy (request/response) | Harder (async)         |

**Rule of thumb:** Use webhooks when consumers need low-latency notifications and can expose a public HTTPS endpoint. Fall back to polling for batch workloads or environments where inbound connections are blocked.

---

## Webhook Registration & Subscription Management

Consumers register a URL + event filter. A well-designed registration API:

```http
POST /webhooks
{
  "url": "https://consumer.example.com/hooks/orders",
  "events": ["order.created", "order.updated", "order.cancelled"],
  "secret": "optional-consumer-provided-secret"
}
→ { "id": "wh_abc123", "status": "active", "signing_secret": "whsec_..." }
```

**Key design decisions:**
- **Event filtering at registration** — consumers subscribe to specific event types, not all events. Reduces noise and payload volume.
- **One URL per webhook vs. one URL per event** — one URL is simpler; per-event URLs allow consumers to route internally.
- **Verification/handshake** — send a `POST` with a `challenge` field on registration; consumer echoes it back to prove ownership (used by Slack, Zoom).
- **Status lifecycle:** `active → paused → disabled`. Auto-pause after N consecutive failures; require manual re-enable.
- **Rate limits per endpoint** — prevent a single consumer's slow endpoint from consuming all delivery threads.

---

## Payload Design: The Envelope Pattern

Wrap every webhook payload in a consistent envelope rather than sending raw resource objects.

```json
{
  "id": "evt_01HXZ9Q...",
  "type": "order.created",
  "api_version": "2026-01-01",
  "created_at": "2026-04-08T18:21:00Z",
  "idempotency_key": "evt_01HXZ9Q...",
  "data": {
    "object": {
      "id": "ord_999",
      "status": "created",
      "amount": 4999,
      "currency": "usd"
    },
    "previous_attributes": {}
  }
}
```

**Why the envelope matters:**
- `type` lets consumers route without parsing `data`.
- `api_version` allows the consumer to handle breaking changes gracefully.
- `id` / `idempotency_key` enables deduplication.
- `previous_attributes` (Stripe pattern) shows what changed on update events — avoids a follow-up GET.

**Avoid:** Sending only an ID and forcing consumers to call back for data ("thin payload"). This creates N+1 API calls and a race condition if the resource changes between delivery and fetch.

---

## Security

### HMAC Signatures
The producer signs each payload with a shared secret using HMAC-SHA256 and sends the signature in a header.

```
X-Hub-Signature-256: sha256=<hex-digest>
```

**Consumer verification (Python):**
```python
import hmac, hashlib

def verify(secret: str, payload: bytes, sig_header: str) -> bool:
    expected = "sha256=" + hmac.new(
        secret.encode(), payload, hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, sig_header)  # constant-time compare
```

**Critical:** Use `hmac.compare_digest` (constant-time) to prevent timing attacks. Never compare strings with `==`.

### Shared Secrets
- Generate per-endpoint secrets (not one global secret) so rotating one doesn't break all consumers.
- Rotate secrets without downtime: send both old and new signatures during a transition window.
- Minimum 32 bytes of entropy; encode as hex or base64.

### IP Allowlisting
Publish a list of source IPs (like Stripe's `stripe.com/files/ips/ips_webhooks.json`). Consumers can firewall all other IPs. Combine with HMAC — IP allowlisting alone is insufficient if your network is shared.

### HTTPS Only
Reject registrations for `http://` URLs. An HTTP endpoint exposes payloads and signatures to MITM.

---

## Delivery Guarantees

Webhooks are inherently **at-least-once**. Exactly-once is impractical without distributed consensus between producer and consumer.

### Producer-Side Retry Strategy
```
Attempt 1:  immediately
Attempt 2:  5s
Attempt 3:  30s
Attempt 4:  2m
Attempt 5:  10m
Attempt 6:  1h
Attempt 7:  6h
Attempt 8+: 24h (up to 72h total, then dead-letter)
```
Jitter each delay by ±10% to avoid thundering herd when a consumer recovers.

**Success criteria:** HTTP 2xx within a timeout (e.g., 30s). Anything else — timeout, 4xx, 5xx — triggers retry. Exception: `410 Gone` should disable the webhook immediately.

### Consumer-Side Idempotency
Every delivery includes a stable `id`. Consumers must deduplicate:

```sql
-- Before processing:
INSERT INTO processed_webhooks (event_id, received_at)
VALUES ($1, NOW())
ON CONFLICT (event_id) DO NOTHING
RETURNING id;
-- If no row returned, skip processing (already handled)
```

Return `200 OK` immediately after enqueuing to a local queue; process asynchronously. This prevents timeouts causing spurious retries.

---

## Failure Handling

### Dead Letter Queues (DLQ)
Events that exhaust all retry attempts move to a DLQ. Store the full envelope, all delivery attempts, and HTTP responses for each.

```
webhook_deliveries
  id, event_id, webhook_id, attempt_count,
  last_error, last_http_status, dlq_at
```

### Manual Retry UI
Expose an admin endpoint or dashboard that lets consumers:
1. View all DLQ events for their webhook.
2. Replay individual events or bulk-replay by time range.
3. See delivery attempt history with request/response bodies.

**Stripe Dashboard** shows every event, its delivery status, and a "Resend" button — the gold standard for developer experience.

### Observability
- Emit metrics: delivery latency, success rate, retry queue depth, DLQ size.
- Alert on: DLQ growth rate, per-endpoint failure rate > threshold, retry queue age > SLA.

---

## Real-World Examples

**Stripe:** Envelope pattern with `type`, signed with `Stripe-Signature` (HMAC-SHA256, supports multiple timestamps to handle clock skew), 72-hour retry window, rich dashboard with replay.

**GitHub:** `X-GitHub-Event` header for event type, `X-Hub-Signature-256` for HMAC, `X-GitHub-Delivery` UUID as idempotency key, per-repo or org-level subscriptions.

**Lessons:**
- Both use thin envelope + rich data (not thin payload).
- Both provide replay/resend tooling.
- Both publish IPs for allowlisting but require HMAC for real security.
- Stripe's tolerance for clock skew (±5 min timestamp window) is underrated — prevents failures from NTP drift.

---

## Trade-offs Summary

| Concern | Recommendation |
|---------|---------------|
| Payload size | Prefer fat payload (full object) over thin (ID only) |
| Security | HMAC + HTTPS mandatory; IP allowlist as defense-in-depth |
| Delivery | At-least-once with idempotent consumers; document this clearly |
| Retry | Exponential backoff with jitter; DLQ after exhaustion |
| Debugging | Store all attempts + responses; provide replay UI |
| Secrets | Per-endpoint, rotatable, minimum 32 bytes entropy |
