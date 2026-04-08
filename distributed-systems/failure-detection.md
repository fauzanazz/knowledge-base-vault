---
title: "Failure Detection"
category: distributed-systems
summary: "Failure detection mechanisms help distributed systems identify when processes have crashed or become unavailable through timeouts, pings, and heartbeats."
sources:
  - raw/articles/_done/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:20:44.871Z
---

# Failure Detection

> Failure detection mechanisms help distributed systems identify when processes have crashed or become unavailable through timeouts, pings, and heartbeats.

# Failure Detection

When a client sends a request to a server and doesn't get a reply back, it's impossible to determine what went wrong. The server might be slow, crashed, or there could be a network issue. Without proper failure detection, clients might wait forever for responses.

## Detection Mechanisms

**Timeouts** - Basic mitigation involves setting timeouts with back-off and retry strategies. However, this is reactive and only detects failures when communication is attempted.

**Ping** - Periodic request from client to server to check if process is available. Allows proactive failure detection.

**Heartbeat** - Periodic request from server to client to indicate that server is still alive. Shifts responsibility to the server to prove its availability.

Pings and heartbeats are commonly used in situations where processes communicate frequently, such as within microservice deployments.

## Challenges

Failure detection faces the fundamental challenge that in an asynchronous network, it's impossible to distinguish between a slow process and a failed one. This leads to the trade-off between:

- **False positives** - Marking healthy but slow processes as failed
- **False negatives** - Failing to detect actual failures quickly

The choice of timeout values and detection frequency must balance these concerns based on system requirements for availability and consistency.

Failure detection is closely related to [[System Models]] and is fundamental to implementing [[System Resiliency]] patterns.

---
*Related: [[System Models]], [[System Resiliency]], [[Distributed Systems]]*
