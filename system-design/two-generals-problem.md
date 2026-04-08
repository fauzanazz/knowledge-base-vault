---
title: "Two Generals Problem"
category: system-design
summary: "The Two Generals Problem illustrates the fundamental challenge of achieving coordination between nodes in distributed systems when communication channels are unreliable and messages can be lost or intercepted."
sources:
  - raw/articles/introduction-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:16:31.621Z
---

# Two Generals Problem

> The Two Generals Problem illustrates the fundamental challenge of achieving coordination between nodes in distributed systems when communication channels are unreliable and messages can be lost or intercepted.

# Two Generals Problem

The **Two Generals Problem** is a classic thought experiment that illustrates the fundamental coordination challenges in [[Distributed Systems]] when communication channels are unreliable.

## The Problem

Two generals need to coordinate a joint attack on a city. They are positioned on opposite sides of the city and can only communicate by sending messengers through enemy territory. The challenge:

- All messages between the generals can be **intercepted and dropped**
- Both generals must attack **at exactly the same time** to succeed
- If only one general attacks, the attack will fail
- How can they ensure synchronized action despite unreliable communication?

## The Impossibility

The Two Generals Problem demonstrates that **perfect coordination is impossible** in distributed systems with unreliable communication channels. Even if a general receives confirmation that their message was received, they cannot be certain that their acknowledgment of that confirmation was received.

This creates an infinite chain of required acknowledgments:
1. General A sends attack time
2. General B confirms receipt
3. General A must confirm the confirmation
4. General B must confirm the confirmation of the confirmation
5. And so on...

## Real-World Implications

This problem reflects real challenges in distributed systems:
- **[[Network Communication]]** can fail at any point
- Perfect consensus is theoretically impossible
- Systems must be designed to handle uncertainty
- Practical solutions involve timeouts, retries, and accepting some risk

## Practical Solutions

While perfect coordination is impossible, distributed systems use various techniques like consensus algorithms, leader election, and eventual consistency to achieve practical coordination despite these fundamental limitations.

---
*Related: [[Distributed Systems]], [[Network Communication]], [[Consensus Algorithms]]*
