---
title: "Event Logs"
category: system-design
summary: "Event logs are immutable, time-stamped records of system events that provide detailed insights into service behavior. They can be structured or unstructured and help with debugging but have significant cost and performance considerations."
sources:
  - raw/articles/maintainability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:15:09.676Z
---

# Event Logs

> Event logs are immutable, time-stamped records of system events that provide detailed insights into service behavior. They can be structured or unstructured and help with debugging but have significant cost and performance considerations.

# Event Logs

Event logs are immutable lists of time-stamped events that record what happened in a system over time. They provide detailed insights into service behavior and are essential for debugging production issues.

## Log Formats

Logs can be formatted as:
- **Free text**: Human-readable but harder to query
- **Structured formats**: JSON or protobuf for better queryability

```json
{
  "failureCount": 1,
  "serviceRegion": "EastUs2",
  "timestamp": 1614438079
}
```

## Advantages and Disadvantages

**Advantages:**
- Simple to emit from applications
- Provide detailed execution traces
- Help debug service crashes and request flows

**Disadvantages:**
- Can add overhead if not flushed asynchronously
- Expensive to store and ingest due to volume
- Low signal-to-noise ratio
- Can fill disk space and degrade service performance

## Best Practices

- **Context tagging**: Tag related events with request IDs for correlation
- **Rich metadata**: Include who, what, success/failure, and duration
- **PII removal**: Strip sensitive information from logs
- **Log levels**: Use ERROR, WARN, INFO levels with configurable thresholds
- **Sampling**: Log every Nth event to control volume
- **Rate limiting**: Prevent log surges from overwhelming collectors

Event logs are typically collected asynchronously by systems like ELK stack or AWS CloudWatch for analysis and debugging.

---
*Related: [[Observability]], [[Distributed Tracing]], [[System Monitoring]], [[Structured Logging]]*
