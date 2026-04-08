---
title: "Vector Clock"
category: data-structures
summary: "Vector clocks are distributed timestamp mechanisms that track causality and detect conflicts in replicated data systems. They use [server, version] pairs to determine if data versions are concurrent or causally related."
sources:
  - raw/articles/key-value-store-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:40:37.040Z
---

# Vector Clock

> Vector clocks are distributed timestamp mechanisms that track causality and detect conflicts in replicated data systems. They use [server, version] pairs to determine if data versions are concurrent or causally related.

# Vector Clock

A **vector clock** is a distributed timestamp mechanism used to track data versions and detect conflicts in replicated systems. It consists of [server, version] pairs associated with each data item.

## Structure

A vector clock is represented as:
```
D([S1, v1], [S2, v2], ..., [Sn, vn])
```

Where:
- `D` is the data item
- `Si` is the server identifier
- `vi` is the version counter for server `Si`

## Operations

### Updating Vector Clocks
When data item D is written to server Si:
- **If Si exists**: Increment its version counter
- **If Si doesn't exist**: Add new [Si, 1] entry to vector clock

### Conflict Detection

**No Conflict (Ancestor Relationship)**:
Version X is an ancestor of version Y if all counters in X are ≤ corresponding counters in Y.

**Conflict Exists (Sibling Relationship)**:
Two versions are siblings if there's at least one counter in each version that is less than its counterpart in the other.

## Example Scenario

1. Server 1 changes name: `D([S1, 1])`
2. Server 2 simultaneously changes name: `D([S2, 1])`
3. Result: Conflicting versions v1 and v2 (siblings)
4. System detects conflict and requires resolution

## Conflict Resolution

When conflicts are detected:
- **Application Logic**: Custom business rules handle conflicts
- **Client Intervention**: Present options to users for manual resolution
- **Last-Writer-Wins**: Simple but potentially lossy approach
- **Merge Strategies**: Combine conflicting values when possible

## Advantages

- **Causality Tracking**: Determines if events are causally related
- **Conflict Detection**: Identifies when data versions diverge
- **Distributed**: Works without central coordination
- **Deterministic**: Same inputs produce same conflict detection results

## Challenges

- **Complexity**: Increases client-side logic requirements
- **Size Growth**: Vector clocks grow with number of servers and updates
- **Trimming Required**: Need strategies to limit vector clock size
- **Storage Overhead**: Additional metadata for each data item

## Applications

Vector clocks are essential in:
- [[Key-Value Store]] systems for conflict resolution
- Distributed databases implementing eventual consistency
- Version control systems
- Collaborative editing applications

They enable [[Database Replication]] systems to maintain consistency while allowing concurrent updates across multiple nodes.

---
*Related: [[Key-Value Store]], [[Database Replication]], [[Distributed Systems]], [[Conflict Resolution]], [[Eventual Consistency]]*
