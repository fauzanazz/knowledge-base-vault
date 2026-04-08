---
title: "Failure Detection"
category: distributed-systems
summary: "Failure detection mechanisms help distributed systems identify when processes have crashed or become unavailable through timeouts, pings, and heartbeats."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.007Z
---

# Failure Detection

> Failure detection mechanisms help distributed systems identify when processes have crashed or become unavailable through timeouts, pings, and heartbeats.

# Failure Detection

When a client sends a request and doesn't receive a response, it's impossible to distinguish between a slow server, a crashed server, or network issues. **Failure detection** mechanisms help identify unavailable processes.

## Detection Strategies

**Timeouts**: Clients wait for a maximum duration before considering a request failed. This requires careful tuning - too short causes false positives, too long delays failure detection.

**Pings**: Periodic requests from clients to servers to check availability. The client actively probes the server's health.

**Heartbeats**: Periodic messages from servers to clients indicating they're still alive. The server actively signals its health status.

## Implementation Considerations

Pings and heartbeats are commonly used when processes communicate frequently, such as within microservice deployments. They help maintain an updated list of available processes.

The challenge is setting appropriate timeouts and intervals. Network delays, garbage collection pauses, and temporary slowdowns can trigger false failure detections, leading to unnecessary failovers.

## Limitations

Failure detection in asynchronous systems is inherently imperfect. The [[System Models|partially synchronous model]] assumes the system behaves predictably most of the time, making failure detection practical despite theoretical impossibility in pure asynchronous systems.

Effective failure detection is crucial for [[Leader Election]] algorithms and maintaining system availability in distributed architectures.

---
*Related: [[System Models]], [[Leader Election]], [[Timeouts]]*
