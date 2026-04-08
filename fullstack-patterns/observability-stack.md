---
title: "Observability Stack"
category: fullstack-patterns
summary: "A production monitoring architecture combining OpenTelemetry instrumentation, Prometheus metrics, Grafana visualization, structured logging, and distributed tracing to achieve full system visibility."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Observability Stack

> A production monitoring architecture combining OpenTelemetry instrumentation, Prometheus metrics, Grafana visualization, structured logging, and distributed tracing to achieve full system visibility.

## The Three Pillars

Observability requires three complementary signals:

| Signal | Tool | Answers |
|--------|------|---------|
| **Metrics** | Prometheus + Grafana | "Is my system healthy? What are the rates?" |
| **Logs** | Structured JSON + Loki/Elasticsearch | "What happened when it broke?" |
| **Traces** | OpenTelemetry + Tempo/Jaeger | "Which service/line of code is slow?" |

Modern observability adds a 4th: **Events** (deployments, config changes that correlate with incidents).

## OpenTelemetry

OpenTelemetry (OTel) is the CNCF standard for instrumentation — vendor-neutral SDK for traces, metrics, and logs.

### Auto-Instrumentation (Node.js)
```typescript
// otel.ts — load BEFORE any other imports
import { NodeSDK } from '@opentelemetry/sdk-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { Resource } from '@opentelemetry/resources';
import { SEMRESATTRS_SERVICE_NAME } from '@opentelemetry/semantic-conventions';

const sdk = new NodeSDK({
  resource: new Resource({ [SEMRESATTRS_SERVICE_NAME]: 'my-api' }),
  traceExporter: new OTLPTraceExporter({
    url: process.env.OTEL_EXPORTER_OTLP_ENDPOINT, // Grafana Tempo, Jaeger, etc.
  }),
  instrumentations: [getNodeAutoInstrumentations()],
  // Auto-instruments: HTTP, Express, PostgreSQL, Redis, gRPC, fetch
});

sdk.start();
```

```typescript
// package.json start script
"start": "node --require ./otel.js dist/index.js"
```

### Manual Spans for Business Logic
```typescript
import { trace, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('my-api');

async function processPayment(orderId: string) {
  return tracer.startActiveSpan('processPayment', async (span) => {
    span.setAttribute('order.id', orderId);
    span.setAttribute('order.amount', order.total);
    
    try {
      const result = await stripeClient.charge(order);
      span.setAttribute('payment.status', result.status);
      span.setStatus({ code: SpanStatusCode.OK });
      return result;
    } catch (err) {
      span.recordException(err as Error);
      span.setStatus({ code: SpanStatusCode.ERROR, message: (err as Error).message });
      throw err;
    } finally {
      span.end();
    }
  });
}
```

### Trace Context Propagation
OTel automatically injects `traceparent` headers on outgoing HTTP requests and extracts them on incoming. This chains spans across services into a single trace:

```
Request → API Gateway → User Service → Order Service → Payment Service
             trace_id: abc123 propagated through all hops
```

## Prometheus Metrics

```typescript
// Custom metrics with prom-client
import { Counter, Histogram, Gauge, register } from 'prom-client';

const httpRequestDuration = new Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5],
});

const activeConnections = new Gauge({
  name: 'websocket_connections_active',
  help: 'Number of active WebSocket connections',
});

const paymentErrors = new Counter({
  name: 'payment_errors_total',
  help: 'Total payment processing errors',
  labelNames: ['provider', 'error_type'],
});

// Middleware
app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    end({ method: req.method, route: req.route?.path ?? 'unknown', status_code: res.statusCode });
  });
  next();
});

// Expose metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.send(await register.metrics());
});
```

**Key metrics to expose** (USE method):
- **Utilization**: CPU, memory, connection pool usage
- **Saturation**: queue depth, request queue length
- **Errors**: 5xx rate, exception count, timeout rate

**RED method** for services:
- **Rate**: requests per second
- **Errors**: error rate
- **Duration**: p50/p95/p99 latency

## Grafana Dashboards

```json
// Prometheus queries for key panels
{
  "httpErrorRate": "sum(rate(http_requests_total{status_code=~'5..'}[5m])) / sum(rate(http_requests_total[5m]))",
  
  "p99Latency": "histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service))",
  
  "requestRate": "sum(rate(http_requests_total[1m])) by (service)",
  
  "apdex": "( sum(rate(http_request_duration_seconds_bucket{le='0.3'}[5m])) + sum(rate(http_request_duration_seconds_bucket{le='1.2'}[5m])) ) / 2 / sum(rate(http_request_duration_seconds_count[5m]))"
}
```

**Grafana stack (LGTM)**:
- **Loki**: log aggregation (label-based, not full-text index like ES — cheaper)
- **Grafana**: visualization for all signals
- **Tempo**: distributed tracing backend
- **Mimir**: long-term Prometheus metrics storage

## Structured Logging

Logs must be structured JSON in production — never `console.log('User logged in: ' + userId)`:

```typescript
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL ?? 'info',
  formatters: {
    level: (label) => ({ level: label }),  // use string, not number
  },
  base: {
    service: 'my-api',
    version: process.env.DEPLOY_SHA,
    env: process.env.NODE_ENV,
  },
});

// Context-aware logger — include trace ID from OTel
function getContextLogger(req: Request) {
  const span = trace.getActiveSpan();
  const traceId = span?.spanContext().traceId;
  
  return logger.child({
    traceId,           // Correlate logs with traces
    userId: req.user?.id,
    requestId: req.headers['x-request-id'],
  });
}

// Usage
const log = getContextLogger(req);
log.info({ orderId, amount }, 'Payment initiated');
log.error({ err, orderId }, 'Payment failed');
```

**Log levels in production**:
- `error`: actionable, pages on-call
- `warn`: degraded but functional
- `info`: business events (order placed, user registered)
- `debug`: disabled in prod; enable per-service dynamically

## Alerting

```yaml
# Prometheus alerting rules
groups:
  - name: api.rules
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m])) 
          / sum(rate(http_requests_total[5m])) > 0.01
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High error rate: {{ $value | humanizePercentage }}"
          runbook: "https://runbooks.example.com/high-error-rate"
      
      - alert: HighLatency
        expr: |
          histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m])) > 1.0
        for: 5m
        labels:
          severity: warning
```

**Alertmanager** routes alerts to PagerDuty/Slack/OpsGenie. Deduplication, inhibition, and grouping prevent alert storms.

## Distributed Tracing Patterns

**Sampling strategies** (trace everything = too expensive):
- **Probabilistic**: sample 1% of all requests
- **Rate limiting**: max 100 traces/sec
- **Tail-based sampling**: sample 100% of errors, 1% of successes (OTel Collector supports this)

```yaml
# OTel Collector tail sampling config
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: errors-policy
        type: status_code
        status_code: { status_codes: [ERROR] }
      - name: slow-traces
        type: latency
        latency: { threshold_ms: 1000 }
      - name: probabilistic
        type: probabilistic
        probabilistic: { sampling_percentage: 1 }
```

## SLOs and Error Budgets

```
SLO: 99.9% of requests complete successfully in <500ms over 30 days
Error budget: 0.1% = 43.2 minutes of downtime per month

Budget burn rate alert: if burning budget 14x faster than allowed → page
```

Define SLOs in Grafana using recording rules; track burn rate to catch slow degradations before SLO breach.

## Trade-offs

| Pro | Con |
|-----|-----|
| Vendor-neutral OTel instrumentation | Significant infra to self-host full LGTM stack |
| Correlate logs, metrics, traces | Cardinality explosions in Prometheus (too many labels) |
| Full system visibility | Sampling means you might miss rare errors |
| OTel auto-instrumentation is low effort | Storage costs for high-volume logs/traces |

## When to Use

✅ Any production service with real users  
✅ Microservices where request paths cross multiple services  
✅ Before adding new features (instrument first)  
✅ After incidents (add instrumentation to gaps found)  
❌ Prototype/MVP with no real traffic (overhead isn't worth it)  
❌ Team without dedicated infra (use managed: Datadog, Honeycomb, New Relic)

---
*Related: [[Canary Deployments]], [[GitOps]], [[WebSocket Architecture]]*
