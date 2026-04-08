---
title: "Shared Nothing Architecture"
category: backend-architecture
summary: "A distributed computing approach where each node is fully independent with its own CPU, memory, and disk — eliminating shared resources to achieve near-linear horizontal scalability and eliminate single points of contention."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Shared Nothing Architecture

> A distributed computing approach where each node is fully independent with its own CPU, memory, and disk — eliminating shared resources to achieve near-linear horizontal scalability and eliminate single points of contention.

## Overview

Coined by Michael Stonebraker in 1986, **Shared Nothing Architecture (SN)** contrasts with Shared Memory (SMP) and Shared Disk architectures. In SN, each node in a cluster has exclusive access to its resources — no network file system, no distributed lock manager, no shared memory. Inter-node communication happens only through explicit message passing.

This principle underlies web scalability: **stateless, horizontally scalable services** that store state externally (database, cache, object storage) are the modern expression of shared nothing.

## Shared Resource Architectures Compared

```
Shared Memory (SMP):
  Node A ──┐
  Node B ──┼──► Shared RAM bus ──► Single CPU pool
  Node C ──┘
  Problem: Memory bus contention; scales to ~64 cores max

Shared Disk:
  Node A ──┐
  Node B ──┼──► Shared SAN / NFS
  Node C ──┘
  Problem: Disk I/O contention; distributed locking overhead

Shared Nothing:
  Node A: [CPU][RAM][Disk A]  ─── Message Passing ───
  Node B: [CPU][RAM][Disk B]  ─────────────────────── Cluster
  Node C: [CPU][RAM][Disk C]  ───────────────────────
  Result: Add nodes → linear throughput increase
```

## Core Principles in Modern Systems

### 1. Stateless Application Servers
Web/API server instances hold **no state** between requests. Session data, user context, and computed results live in external systems:

```python
# ❌ Stateful server — cannot horizontally scale
class UserService:
    _sessions = {}  # in-memory — lost on restart, not shared across instances

    def login(self, user_id):
        self._sessions[user_id] = {"logged_in": True, "cart": []}
```

```python
# ✅ Shared Nothing — state in Redis
class UserService:
    def __init__(self, redis_client):
        self.redis = redis_client

    def login(self, user_id):
        session_data = {"logged_in": True, "cart": []}
        self.redis.setex(f"session:{user_id}", 3600, json.dumps(session_data))
        # Any instance can serve subsequent requests for this user
```

### 2. Data Partitioning (Sharding)
Data is horizontally partitioned across nodes. Each partition (shard) is owned by one node — no cross-node locking:

```
User data partitioned by user_id hash:
  Shard 0: user_id % 4 == 0  → Node A
  Shard 1: user_id % 4 == 1  → Node B
  Shard 2: user_id % 4 == 2  → Node C
  Shard 3: user_id % 4 == 3  → Node D

Request for user_id = 12345:
  12345 % 4 = 1 → Route to Node B
  Node B handles entirely — no coordination with A/C/D
```

### 3. Consistent Hashing (for Dynamic Clusters)
Adding/removing nodes redistributes minimal data:

```python
import hashlib
from sortedcontainers import SortedList

class ConsistentHashRing:
    def __init__(self, nodes=None, replicas=150):
        self.ring = {}
        self.sorted_keys = SortedList()
        self.replicas = replicas
        for node in (nodes or []):
            self.add_node(node)

    def add_node(self, node):
        for i in range(self.replicas):
            key = self._hash(f"{node}:{i}")
            self.ring[key] = node
            self.sorted_keys.add(key)

    def get_node(self, key):
        hash_key = self._hash(key)
        idx = self.sorted_keys.bisect_left(hash_key) % len(self.sorted_keys)
        return self.ring[self.sorted_keys[idx]]

    def _hash(self, key):
        return int(hashlib.md5(key.encode()).hexdigest(), 16)
```

## Database Implementations

### PostgreSQL Citus (Shared Nothing Sharding)
```sql
-- Citus: distribute orders table across nodes by customer_id
SELECT create_distributed_table('orders', 'customer_id');

-- Queries automatically route to the correct shard
SELECT * FROM orders WHERE customer_id = 'cust_123';
-- ↑ Executes only on the node owning customer_id hash shard
```

### Cassandra (True Shared Nothing)
Cassandra is a pure Shared Nothing system — no primary node, no shared storage. Every node is identical:

```
cassandra.yaml:
  num_tokens: 256  # virtual nodes per physical node
  partitioner: org.apache.cassandra.dht.Murmur3Partitioner

Write path:
  Client → any Cassandra node (coordinator)
  Coordinator → hash(partition_key) → responsible nodes
  No single node coordinates all writes
```

### Vitess (YouTube's MySQL Sharding)
YouTube/Slack use Vitess to scale MySQL horizontally with shared-nothing sharding across thousands of nodes.

## Stateless Services in Kubernetes

```yaml
# Kubernetes Deployment — stateless, shared nothing
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-service
spec:
  replicas: 50       # 50 identical, independent instances
  template:
    spec:
      containers:
      - name: api
        image: api-service:v2.1
        env:
        - name: SESSION_STORE
          value: "redis://redis-cluster:6379"  # external state
        - name: DB_HOST
          value: "postgres-cluster:5432"
        resources:
          requests: { cpu: "500m", memory: "512Mi" }
```

Any of the 50 pods can handle any request — fully interchangeable.

## Content Distribution: CDN as Shared Nothing

CDNs implement shared nothing at the global level:
- Each edge node (PoP) serves requests entirely from its own local cache
- No cross-PoP coordination for cache hits
- Origin pull is the only "shared" resource, but it's only for cache misses

Cloudflare, Fastly, and Akamai serve trillions of requests this way.

## Real-World Examples

**Amazon Web Services (2000s)**: Jeff Bezos's "Two Pizza Teams" and API mandate forced shared-nothing service boundaries — teams couldn't share internal resources, only communicate through explicit APIs. This is the organizational expression of shared nothing.

**Facebook TAO (Social Graph)**: Horizontally sharded across thousands of MySQL instances with a shared-nothing in-memory cache layer. Each shard is independent — social graph queries fan out to the relevant shards.

**Twitter (2014 rewrite)**: Moved from a shared Ruby on Rails monolith to shared-nothing microservices. Each service (Tweets, Users, Timeline) independently sharded, no shared databases.

**Google Spanner**: Achieves global consistency across shared-nothing shards using TrueTime (atomic clock synchronization) — proving shared nothing can coexist with strong consistency at global scale.

**Memcached/Redis Cluster**: Both use consistent hashing to distribute keys across nodes with no inter-node coordination for reads/writes.

## The CAP Theorem Intersection

Shared Nothing architectures must make CAP tradeoffs:
```
Partition Tolerance (P) is mandatory in distributed shared nothing systems

Choose:
  CP: Consistent + Partition Tolerant → sacrifice availability on partition
      (HBase, Zookeeper, Etcd)
  AP: Available + Partition Tolerant → sacrifice consistency on partition
      (Cassandra, DynamoDB, CouchDB)
```

## Trade-offs

| Pros | Cons |
|------|------|
| Near-linear horizontal scalability | Cross-node operations (distributed JOINs) are expensive |
| No single point of contention | Resharding is operationally complex |
| Fault isolation — one node failure is contained | Eventual consistency in AP systems requires careful design |
| Independent node upgrade/replacement | No cross-node transactions without distributed protocols |
| Cost-efficient: commodity hardware | Application must be explicitly designed for it |

## Anti-Patterns

- **Sticky sessions**: Pinning users to specific nodes creates implicit state sharing and uneven load
- **Distributed locking**: Using Zookeeper or Redis locks to coordinate between nodes — reintroduces contention
- **Cross-shard JOINs**: Scatter-gather queries across all shards for unpartitioned data — destroys scalability
- **Server-side session state**: In-memory session → can't route to different instance → no horizontal scale

## When to Use

✅ **Use when**: High-traffic web services requiring horizontal scaling; read-heavy workloads that partition naturally; stateless API layers; distributed databases/caches.

❌ **Avoid when**: Operations requiring strong consistency across many records; small-scale systems where distributed complexity isn't worth the operational overhead; workflows with complex multi-entity transactions.

---
*Related: [[Database per Service]], [[Actor Model]], [[Bulkhead Pattern]], [[Serverless Architecture]]*
