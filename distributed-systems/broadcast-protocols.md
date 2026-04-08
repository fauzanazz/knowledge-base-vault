---
title: "Broadcast Protocols"
category: distributed-systems
summary: "Broadcast protocols enable message delivery to multiple receivers in distributed systems, with different guarantees for reliability and ordering."
sources:
  - raw/articles/coordination-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:42:20.013Z
---

# Broadcast Protocols

> Broadcast protocols enable message delivery to multiple receivers in distributed systems, with different guarantees for reliability and ordering.

# Broadcast Protocols

**Broadcast protocols** implement multicast communication over unicast networks like the internet, enabling message delivery to groups of receivers with various reliability and ordering guarantees.

## Protocol Types

**Best-effort broadcast** guarantees message delivery to all non-faulty receivers. Implementation involves sending messages to all receivers over reliable links, but sender failures can prevent complete delivery.

**Reliable broadcast** ensures eventual delivery to all non-faulty processes even if the sender crashes mid-transmission. **Eager reliable broadcast** has every receiver retransmit messages to the group, requiring N² transmissions.

**Gossip broadcast** optimizes by sending messages to random subsets of nodes. This probabilistic approach doesn't guarantee delivery but minimizes non-reception probability through configuration tuning. Particularly useful for large-scale systems where deterministic protocols don't scale.

**Total order broadcast** guarantees both message delivery and ordering, but requires consensus, creating a scalability bottleneck.

## Implementation Challenges

The main challenge is supporting multiple senders and receivers that can crash at any point while maintaining delivery guarantees.

Reliable broadcast protocols ensure message delivery but don't provide ordering guarantees. Adding ordering requires consensus mechanisms, which limit scalability.

## Use Cases

Broadcast protocols are fundamental building blocks for:
- State machine replication
- Distributed coordination
- Event dissemination in large-scale systems

The choice between protocols depends on the specific consistency, performance, and scalability requirements of the application.

---
*Related: [[Consensus]], [[State Machine Replication]], [[Distributed Systems]]*
