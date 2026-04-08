# Redis: In-Depth Architecture & Use Cases

> **Redis** (Remote Dictionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine. Its combination of extreme speed, rich data types, and operational simplicity makes it one of the most deployed pieces of infrastructure in modern system design.

---

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Core Data Structures](#core-data-structures)
3. [Persistence: RDB & AOF](#persistence-rdb--aof)
4. [Replication](#replication)
5. [High Availability: Redis Sentinel](#high-availability-redis-sentinel)
6. [Horizontal Scale: Redis Cluster](#horizontal-scale-redis-cluster)
7. [Use Cases](#use-cases)
8. [Redis vs Memcached](#redis-vs-memcached)
9. [Redis vs DynamoDB for Caching](#redis-vs-dynamodb-for-caching)
10. [Redis Streams vs Kafka](#redis-streams-vs-kafka)
11. [When NOT to Use Redis](#when-not-to-use-redis)

---

## Architecture Overview

### Single-Threaded Event Loop

Redis's most distinctive architectural choice is its **single-threaded command-processing model**. All client commands are processed sequentially by one main thread, eliminating the need for locks, mutexes, or complex concurrency primitives around data structures.

```
┌──────────────────────────────────────────────────────┐
│                   Redis Process                       │
│                                                       │
│  ┌─────────────┐    ┌──────────────────────────────┐ │
│  │  epoll/kqueue│───▶│  Event Loop (ae.c)           │ │
│  │  I/O Mux    │    │  - Accept connections         │ │
│  └─────────────┘    │  - Read commands              │ │
│                     │  - Execute commands           │ │
│  ┌─────────────┐    │  - Write responses            │ │
│  │  Background  │    └──────────────────────────────┘ │
│  │  Threads:    │                                      │
│  │  - AOF fsync │    ┌──────────────────────────────┐ │
│  │  - RDB fork  │    │  In-Memory Data Store        │ │
│  │  - Lazy free │    │  (Hash tables, skiplists…)   │ │
│  └─────────────┘    └──────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

**Why single-threaded is fast:**
- No context-switching overhead between threads
- CPU cache locality is maximized — all data accessed by one thread
- Network I/O is the bottleneck, not CPU; a single thread can easily saturate 1–10 Gbps links
- Simple mental model; no race conditions in user-space data structures

**What runs in background threads (Redis 4.0+):**
- AOF `fsync` calls (I/O intensive, safe to offload)
- Large key deletion (`UNLINK` / lazy-free)
- RDB child process (via `fork()`, separate process not thread)

**Redis 6.0 I/O Threading:** Redis optionally multi-threads *network I/O* (reading client buffers, writing responses) while keeping *command execution* single-threaded. This bridges the gap for high-connection-count workloads.

### Memory Management

Redis uses **jemalloc** as its allocator and tracks memory usage precisely. The `maxmemory` directive caps consumption; when hit, an eviction policy kicks in (`allkeys-lru`, `volatile-ttl`, `allkeys-random`, etc.). Redis's internal encoding automatically selects compact representations (e.g., ziplist → hashtable) based on element count/size thresholds.

---

## Core Data Structures

| Type | Commands | Internal Encoding | Typical Use |
|------|----------|-------------------|-------------|
| **String** | GET, SET, INCR, APPEND | raw, embstr, int | Counters, cached values |
| **List** | LPUSH, RPOP, LRANGE | listpack, quicklist | Queues, activity feeds |
| **Hash** | HSET, HGET, HMGET | listpack, hashtable | Object fields, sessions |
| **Set** | SADD, SMEMBERS, SINTER | listpack, hashtable | Tags, unique visitors |
| **Sorted Set** | ZADD, ZRANGE, ZRANGEBYSCORE | listpack, skiplist+hashtable | Leaderboards, scheduled jobs |
| **Stream** | XADD, XREAD, XACK | radix tree + listpack | Event logs, message queues |
| **Bitmap** | SETBIT, GETBIT, BITCOUNT | String | Feature flags, daily active users |
| **HyperLogLog** | PFADD, PFCOUNT | String | Approximate cardinality |
| **Geo** | GEOADD, GEODIST, GEORADIUS | Sorted Set | Location-based queries |

---

## Persistence: RDB & AOF

Redis is in-memory first, but offers two persistence mechanisms that can be used independently or together.

### RDB (Redis Database Snapshot)

RDB produces **point-in-time binary snapshots** of the dataset via `BGSAVE`. Redis calls `fork()`, and the child process writes a compact `.rdb` file while the parent continues serving requests (Copy-on-Write means pages are only duplicated when modified).

```
# redis.conf
save 900 1       # snapshot if ≥1 key changed in 900s
save 300 10      # snapshot if ≥10 keys changed in 300s
save 60 10000    # snapshot if ≥10000 keys changed in 60s
```

**Pros:** Compact file, fast restart, minimal runtime overhead.  
**Cons:** Data loss between last snapshot and crash (RPO = minutes).

### AOF (Append-Only File)

AOF logs every write command as it arrives. On restart, Redis replays the log to reconstruct state. Three fsync policies:

| Policy | Durability | Performance |
|--------|-----------|-------------|
| `always` | Every command fsynced | Slowest (~1k ops/s) |
| `everysec` | fsync once per second | Balanced (default) |
| `no` | OS decides when to flush | Fastest, worst durability |

**AOF Rewrite:** Periodically, Redis rewrites the AOF into an equivalent minimal set of commands, shrinking the file.

### Combining RDB + AOF

The recommended production setup: AOF for durability (`everysec`), RDB for fast restores. On startup, if AOF is enabled Redis prefers it (more complete); RDB is kept for disaster recovery.

```
appendonly yes
appendfsync everysec
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

---

## Replication

Redis uses **asynchronous primary→replica replication** (also called leader→follower).

```
           Primary
          /   |   \
    Replica  Replica  Replica
```

- Replicas connect to the primary and receive a **full sync** (RDB transfer) initially
- Subsequent changes are streamed via **replication backlog** (partial resync on reconnect if lag is within buffer)
- Replicas are **read-only** by default — useful for scaling read-heavy workloads
- `WAIT numreplicas timeout` command blocks until N replicas acknowledge writes (synchronous promotion path)

**Replication lag** is the main concern; reads from replicas may be stale. Use replicas for analytics/reporting queries, not for fresh reads in critical paths.

---

## High Availability: Redis Sentinel

**Redis Sentinel** is the built-in HA solution for primary/replica setups without sharding. Sentinel processes form a quorum and collectively:

1. **Monitor** primary and replica health (PING every second)
2. **Alert** on failures via pub/sub notifications
3. **Failover** — promote a replica to primary when quorum agrees the primary is down
4. **Config provider** — clients query Sentinel for the current primary address

```
┌──────────┐    ┌──────────┐    ┌──────────┐
│Sentinel 1│    │Sentinel 2│    │Sentinel 3│
└────┬─────┘    └────┬─────┘    └────┬─────┘
     │               │               │
     └───────────────┼───────────────┘
                     │ monitors
              ┌──────┴──────┐
              │   Primary   │────▶ Replica A
              └─────────────┘────▶ Replica B
```

**Minimum deployment:** 3 Sentinel nodes (quorum = 2) to avoid split-brain. Sentinel handles automatic failover but does **not** provide sharding — the dataset must fit on a single node.

---

## Horizontal Scale: Redis Cluster

Redis Cluster distributes data across multiple nodes using **hash slots** — 16,384 total slots, each assigned to a primary shard.

```
Slot allocation example (3 shards):
  Node A: slots 0–5460
  Node B: slots 5461–10922
  Node C: slots 10923–16383

Key → CRC16(key) % 16384 → slot → node
```

Each primary shard can have replicas for HA. Cluster performs automatic failover (no external Sentinel needed).

**Hash tags:** `{user}.profile` and `{user}.session` hash on `user`, ensuring they land on the same slot — required for multi-key operations.

**Limitations of Redis Cluster:**
- Cross-slot multi-key operations (`MGET`, `SUNION`) require keys to share a hash tag
- Lua scripts must touch only keys on a single slot
- Cluster topology changes (resharding) require careful orchestration

**When to use Cluster vs Sentinel:** Use Sentinel for datasets < single-node capacity with HA requirements. Use Cluster when you need to scale beyond a single node's RAM (typically > 100–200 GB).

---

## Use Cases

### 1. Caching

The canonical Redis use case. Store frequently-accessed results (DB queries, rendered pages, API responses) with a TTL.

```
// Cache-aside pattern
value = redis.get("product:123")
if value == null:
    value = db.query("SELECT * FROM products WHERE id=123")
    redis.setex("product:123", 3600, serialize(value))
return value
```

**Best practices:**
- Set appropriate TTLs — never cache indefinitely unless data is truly static
- Use `allkeys-lru` eviction for pure caches (any key can be evicted)
- Namespace keys: `app:env:type:id` (e.g., `myapp:prod:user:42`)
- Cache stampede protection: use probabilistic early recomputation or a distributed lock

### 2. Session Store

HTTP sessions stored in Redis are accessible across all application servers, solving the sticky-session problem.

```
SET session:{token} {user_json} EX 1800   # 30-min TTL, refreshed on activity
```

Benefits: horizontal app scaling, TTL-based auto-expiry, instant session invalidation (just `DEL session:{token}`).

### 3. Pub/Sub

Redis Pub/Sub provides a fire-and-forget messaging channel. Publishers send to a channel; all current subscribers receive it.

```
SUBSCRIBE notifications:user:42
PUBLISH notifications:user:42 '{"type":"new_message","from":"alice"}'
```

**Limitations:** No message persistence, no consumer groups, no replay. If a subscriber is offline, messages are lost. For durable messaging, use Redis Streams or Kafka.

### 4. Rate Limiting

**Fixed window** with INCR:
```
key = "rate:{user_id}:{minute_bucket}"
count = INCR key
EXPIRE key 60
if count > LIMIT: reject
```

**Sliding window** with Sorted Sets:
```
ZADD rate:{user_id} {now} {request_id}
ZREMRANGEBYSCORE rate:{user_id} 0 {now - 60s}
count = ZCARD rate:{user_id}
EXPIRE rate:{user_id} 60
```

The Sorted Set approach is more accurate but uses more memory. For high-scale rate limiting, the **token bucket** or **GCRA** algorithms can also be implemented with atomic Lua scripts.

### 5. Leaderboards

Sorted Sets are purpose-built for leaderboards:
```
ZADD game:leaderboard 9850 "alice"
ZADD game:leaderboard 10200 "bob"
ZREVRANK game:leaderboard "alice"   → rank 1 (0-indexed)
ZREVRANGE game:leaderboard 0 9      → top 10 players
ZINCRBY game:leaderboard 150 "alice" → atomic score update
```

Time complexity: O(log N) for add/update, O(log N + M) for range queries — handles millions of players efficiently.

### 6. Distributed Locks

**Redlock** algorithm (single node simplified):
```
SET lock:{resource} {unique_token} NX PX 30000
# NX = only if not exists, PX = expire in 30s
```

Release only if you own the lock (atomic Lua):
```lua
if redis.call("get", KEYS[1]) == ARGV[1] then
    return redis.call("del", KEYS[1])
else
    return 0
end
```

For the full multi-node **Redlock** algorithm: acquire the lock on N/2+1 nodes within the validity time; lock is granted only if a majority succeed. Use `redlock` client libraries rather than implementing from scratch.

---

## Redis vs Memcached

| Dimension | Redis | Memcached |
|-----------|-------|-----------|
| **Data types** | Strings, Lists, Hashes, Sets, Sorted Sets, Streams, HLL, Geo | Strings only |
| **Persistence** | RDB + AOF optional | None (pure cache) |
| **Replication** | Built-in | Not built-in |
| **Clustering** | Built-in cluster + Sentinel | Client-side sharding only |
| **Pub/Sub** | Yes | No |
| **Scripting** | Lua scripts | No |
| **Memory efficiency** | Good, but overhead per key | Slightly more efficient for pure string caching |
| **Multi-threading** | Single-threaded (I/O multi-threaded in v6+) | True multi-threaded |
| **Max value size** | 512 MB | 1 MB |

**Choose Memcached when:**
- You need a dead-simple, ultra-low-latency string cache
- You want true multi-threading to utilize all CPU cores without any config
- Your dataset is larger than RAM and you need simple LRU across a flat keyspace

**Choose Redis when:**
- You need more than string caching (sorted sets, streams, pub/sub)
- You need persistence or HA with automatic failover
- You want a single tool to serve multiple infrastructure roles

In practice, Redis has largely replaced Memcached in new systems because its feature richness rarely costs meaningfully in performance.

---

## Redis vs DynamoDB for Caching

| Dimension | Redis (ElastiCache) | DynamoDB (DAX) |
|-----------|---------------------|----------------|
| **Latency** | Sub-millisecond (µs) | Single-digit ms (DAX), 1–10ms (DynamoDB) |
| **Data model** | Flexible (multiple types) | Key-value / document |
| **TTL support** | Native via EXPIRE | Native TTL attribute |
| **Operational complexity** | Moderate (cluster management) | Low (fully serverless) |
| **Cost model** | Instance-based (predictable) | Pay-per-request or provisioned |
| **Max item size** | 512 MB | 400 KB |
| **Query flexibility** | Rich (sorted sets, scripting) | Limited to key/index lookups |
| **Multi-region** | Manual replication | Global Tables built-in |

**Use DynamoDB as a cache** when:
- You're all-in on AWS serverless (Lambda + DynamoDB)
- You need multi-region replication without operational overhead
- Cache items naturally fit DynamoDB's 400 KB limit and simple key-value access pattern
- You want to avoid managing Redis clusters

**Use Redis for caching** when:
- Sub-millisecond latency is required (Redis is ~10x faster than DAX)
- You need rich querying (leaderboards, range queries, pub/sub)
- Your architecture already includes Redis for other purposes
- You need fine-grained eviction policies and memory control

---

## Redis Streams vs Kafka

| Dimension | Redis Streams | Apache Kafka |
|-----------|---------------|--------------|
| **Durability** | Configurable (AOF/RDB) | Always durable (log-based) |
| **Retention** | Memory-bound or count/size limited | Disk-based, configurable retention (days/weeks) |
| **Throughput** | Hundreds of thousands msg/s | Millions msg/s per partition |
| **Consumer groups** | Yes (XREADGROUP, XACK) | Yes (consumer groups) |
| **Ordering** | Per-stream (single partition) | Per-partition |
| **Replay** | Yes (message IDs allow seek) | Yes (offset-based) |
| **Operational complexity** | Low (part of Redis) | High (ZooKeeper/KRaft, brokers, topics) |
| **Ecosystem** | Limited | Rich (Kafka Connect, Streams, ksqlDB) |

**Use Redis Streams when:**
- Message volume is moderate (< 1M msg/s)
- You already run Redis and want to avoid another infrastructure component
- Messages don't need to be retained longer than your Redis memory/disk allows
- Use case: task queues, activity feeds, real-time notifications, IoT sensor data

**Use Kafka when:**
- You need guaranteed multi-day/week message retention
- You need very high throughput (millions/sec)
- Multiple independent consumer teams need to replay the same stream
- You need rich stream processing (Kafka Streams, ksqlDB)
- Use case: event sourcing, audit logs, data pipelines, analytics ingestion

---

## When NOT to Use Redis

Despite its versatility, Redis is the wrong tool in several scenarios:

### 1. Primary Database for Complex Queries
Redis has no JOINs, no ad-hoc querying, no full-text search beyond basic pattern matching. If your access patterns require SQL-style aggregations or multi-dimensional filtering, use PostgreSQL or another RDBMS.

### 2. Data That Exceeds Available RAM
Redis is fundamentally an in-memory store. While RDB/AOF provide persistence, the working dataset must fit in RAM. At terabyte scale, Redis becomes prohibitively expensive. Consider Cassandra, Aerospike, or column stores.

### 3. Long-Term Durable Message Queuing
Redis Pub/Sub has no durability. Even Redis Streams lose messages if `maxmemory` eviction kicks in and deletes stream entries. For guaranteed, durable, high-volume event streaming, use Kafka or cloud equivalents (Kinesis, Pub/Sub).

### 4. Strong Consistency Requirements
Redis replication is **asynchronous** — replicas may lag. Redis Cluster's gossip-based topology can create brief split-brain windows. If you need linearizable reads (every read reflects the latest write), Redis is not suitable without careful design (e.g., always read from primary only).

### 5. Large Binary Objects (BLOBs)
Storing images, videos, or large files in Redis wastes expensive RAM. Use object storage (S3, GCS) and store only metadata/URLs in Redis.

### 6. GDPR/Compliance Scenarios Requiring Encryption at Rest
Redis open-source does not natively encrypt data at rest (Redis Enterprise does). If regulatory requirements mandate encryption at rest and you can't use managed services (AWS ElastiCache with encryption), evaluate other stores.

### 7. Cost-Sensitive Low-QPS Workloads
For small applications with low traffic, Redis adds operational overhead without proportional benefit. A simple database with query caching or a lightweight local cache (e.g., in-process LRU map) may suffice.

---

## Quick Decision Guide

```
Need sub-ms caching with rich data types?     → Redis
Need simple string caching, multi-core scale? → Memcached (or Redis v6+ I/O threads)
Need fully managed, serverless caching on AWS?→ DynamoDB/DAX
Need durable, high-throughput event streaming?→ Kafka
Need lightweight event streaming, already on Redis? → Redis Streams
Need HA with auto-failover, single dataset?   → Redis Sentinel
Need horizontal scale beyond 1 node?          → Redis Cluster
Need complex queries, JOINs, SQL?             → PostgreSQL/MySQL
Data > available RAM budget?                  → Cassandra, ScyllaDB, or disk-backed stores
```

---

## Further Reading

- [Redis Official Documentation](https://redis.io/docs/)
- [Redis University](https://university.redis.com/) — free courses
- [Redis Persistence Demystified](https://redis.io/docs/management/persistence/)
- [Salvatore Sanfilippo — Redis Design Decisions](http://antirez.com/news/138) (antirez blog)
- [The Redlock Algorithm](https://redis.io/docs/manual/patterns/distributed-locks/)
- Martin Kleppmann's critique of Redlock: [How to do distributed locking](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
