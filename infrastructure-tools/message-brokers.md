# Message Brokers: A Comprehensive Guide

> **Last Updated:** April 2026  
> **Tags:** `messaging`, `distributed-systems`, `kafka`, `rabbitmq`, `event-streaming`, `queues`, `infrastructure`

---

## Table of Contents

1. [Introduction](#introduction)
2. [Core Concepts](#core-concepts)
3. [RabbitMQ](#rabbitmq)
4. [Apache Kafka](#apache-kafka)
5. [RabbitMQ vs. Kafka: Head-to-Head](#rabbitmq-vs-kafka-head-to-head)
6. [Other Notable Brokers](#other-notable-brokers)
7. [Decision Framework](#decision-framework)
8. [Common Patterns & Anti-Patterns](#common-patterns--anti-patterns)
9. [Summary Reference Card](#summary-reference-card)

---

## Introduction

A **message broker** is middleware that enables applications, systems, and services to communicate with each other by translating messages between formal messaging protocols. Rather than services calling each other directly (tight coupling), they publish messages to a broker, which routes, stores, and delivers those messages to the appropriate consumers (loose coupling).

### Why Use a Message Broker?

| Problem | Solution |
|---|---|
| Services are tightly coupled | Producers don't know who consumes their messages |
| Downstream service is slow | Queue absorbs backpressure; producer isn't blocked |
| Traffic spikes overwhelm consumers | Broker buffers messages; consumers process at their own pace |
| Need to fan-out to multiple recipients | One publish → many subscribers |
| Need audit log of all events | Persistent, ordered log (Kafka-style) |
| Asynchronous workflows | Decouple trigger from processing |

### The Messaging Landscape

```
┌─────────────────────────────────────────────────────────┐
│                   MESSAGE BROKERS                        │
├─────────────────────┬───────────────────────────────────┤
│  Traditional Queue  │     Event Streaming Platform       │
│  (Smart Broker)     │     (Dumb Broker, Smart Client)    │
├─────────────────────┼───────────────────────────────────┤
│  RabbitMQ           │  Apache Kafka                      │
│  ActiveMQ           │  Apache Pulsar                     │
│  Amazon SQS         │  AWS Kinesis                       │
│  NATS               │                                    │
└─────────────────────┴───────────────────────────────────┘
```

---

## Core Concepts

Before diving into specific technologies, let's establish shared vocabulary:

- **Producer / Publisher**: Application that sends messages
- **Consumer / Subscriber**: Application that receives messages
- **Queue**: Buffer holding messages for point-to-point delivery
- **Topic**: Named channel for publish/subscribe patterns
- **Acknowledgement (ACK)**: Consumer signals it processed a message successfully
- **Dead Letter Queue (DLQ)**: Where undeliverable or failed messages land
- **At-most-once**: Message may be lost; never delivered twice
- **At-least-once**: Message always delivered; may be duplicated
- **Exactly-once**: Message delivered precisely one time (hardest to achieve)
- **Backpressure**: Mechanism to slow producers when consumers fall behind

---

## RabbitMQ

### Overview

RabbitMQ is an open-source, **general-purpose message broker** originally released in 2007 by Rabbit Technologies (later acquired by Pivotal, now VMware/Broadcom). It implements the **Advanced Message Queuing Protocol (AMQP 0-9-1)** and is renowned for its rich routing capabilities, reliability features, and operational maturity.

Written in **Erlang**, RabbitMQ inherits Erlang's legendary fault-tolerance and concurrency model — the same runtime that powers Ericsson's telecom switches designed for "nine nines" (99.9999999%) uptime.

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    RabbitMQ Broker                       │
│                                                          │
│  Producer → [Exchange] → (Binding) → [Queue] → Consumer  │
│               │                         │                │
│               │  fanout/direct/topic/   │  prefetch,     │
│               │  headers routing        │  ACK, DLQ      │
│                                                          │
│  ┌──────────┐  ┌───────────────────────────────────┐    │
│  │ vHost A  │  │ vHost B (logical isolation)        │    │
│  └──────────┘  └───────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

**Key architectural components:**

- **Virtual Hosts (vHosts)**: Logical namespaces — like databases in MySQL. Different applications can share one broker with complete isolation.
- **Connection & Channel**: Clients open a TCP connection, then create lightweight channels (multiplexed) within it. This avoids the overhead of a new TCP connection per operation.
- **Exchange**: Receives messages from producers and routes them to queues using rules called *bindings*.
- **Queue**: An ordered buffer of messages waiting for consumers.
- **Binding**: A rule linking an exchange to a queue, optionally with a routing key or argument pattern.

### AMQP Protocol

**AMQP (Advanced Message Queuing Protocol)** is a binary, application-layer protocol designed for message-oriented middleware. It defines:

- **Wire-level protocol**: Any AMQP-compliant client in any language can talk to any AMQP broker.
- **Framing**: Messages are broken into frames — method frames (commands), header frames (metadata), body frames (payload).
- **Flow control**: Built-in publisher confirms and consumer ACKs.
- **Transactions**: AMQP supports transactional publishing (though publisher confirms are preferred for performance).

RabbitMQ also supports additional protocols via plugins: **MQTT** (IoT devices), **STOMP** (text-based, web clients), and **HTTP API** (management and publishing).

### Exchange Types

| Exchange Type | Routing Logic | Use Case |
|---|---|---|
| **Direct** | Routes to queues where binding key = routing key | Task queues, specific routing |
| **Fanout** | Broadcasts to ALL bound queues (ignores routing key) | Notifications, cache invalidation |
| **Topic** | Wildcard matching on routing key (`*` = one word, `#` = zero or more) | Flexible pub/sub, log routing |
| **Headers** | Routes based on message header attributes | Complex routing without key |
| **Default** | Routes directly to queue named by routing key | Simple point-to-point |

**Topic Exchange Example:**
```
Routing Key: "payment.credit.failed"

Queue A binds: "payment.#"         → MATCHES (gets message)
Queue B binds: "payment.*.failed"  → MATCHES (gets message)
Queue C binds: "shipment.#"        → NO MATCH
Queue D binds: "#.failed"          → MATCHES (gets message)
```

### Key Features

**Reliability Mechanisms:**
- **Publisher Confirms**: Broker ACKs the producer after writing to disk
- **Consumer ACKs**: Message stays in queue until consumer explicitly acknowledges
- **Persistent Messages**: `delivery_mode=2` writes messages to disk before ACKing
- **Durable Queues**: Queues survive broker restarts
- **Dead Letter Exchanges (DLX)**: Failed messages routed to a DLX for inspection

**Operational Features:**
- **Management UI**: Built-in web dashboard at port 15672 for monitoring queues, connections, rates
- **Shovel & Federation**: Move or mirror messages between brokers/data centers
- **Clustering**: Multiple nodes share queue definitions; queues can be replicated via **Quorum Queues** (Raft-based) or the older Classic Mirrored Queues
- **Priority Queues**: Messages can have 1–255 priority levels
- **TTL**: Per-message or per-queue time-to-live
- **Lazy Queues**: Immediately write messages to disk to handle long queues without RAM exhaustion

### RabbitMQ Use Cases

✅ **Great fit for:**
- **Task queues / work queues**: Distributing jobs across worker pools (image processing, email sending, PDF generation)
- **Complex routing**: Message needs to be routed to different queues based on content/type
- **Request-reply patterns (RPC)**: Correlation IDs and reply-to queues enable synchronous-feeling async RPC
- **Per-message routing control**: When different messages need different handling rules
- **Low-latency requirements**: Optimized to deliver and delete messages quickly
- **Language-agnostic integrations**: Rich client library ecosystem (Python, Java, Go, Ruby, .NET, PHP, etc.)

### RabbitMQ Performance Profile

- **Latency**: Sub-millisecond to single-digit milliseconds for in-memory queues
- **Throughput**: ~50,000–100,000 messages/second on typical hardware (higher with tuning)
- **Message size**: Optimized for small-to-medium messages; large messages should be referenced (store in S3, send URL)
- **Persistence cost**: Disk persistence adds latency; use carefully for high-throughput paths

---

## Apache Kafka

### Overview

Apache Kafka was created at **LinkedIn** in 2010 to handle their activity tracking pipeline (page views, clicks, searches) and open-sourced the same year. It joined the Apache Software Foundation in 2012. Unlike traditional message queues, Kafka is fundamentally a **distributed, append-only commit log** — a persistent, ordered, replayable stream of records.

Where RabbitMQ is a "smart broker / dumb consumer" (broker does complex routing), Kafka is a "dumb broker / smart consumer" (broker stores logs; consumers track their own position).

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Kafka Cluster                            │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │  Topic: "orders"                                         │   │
│  │                                                          │   │
│  │  Partition 0: [msg0][msg1][msg4][msg7] → offset: 3      │   │
│  │  Partition 1: [msg2][msg3][msg5][msg8] → offset: 3      │   │
│  │  Partition 2: [msg6][msg9][...]        → offset: 1      │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐                   │
│  │ Broker 1 │   │ Broker 2 │   │ Broker 3 │                   │
│  │(Leader P0)│   │(Leader P1)│   │(Leader P2)│                   │
│  └──────────┘   └──────────┘   └──────────┘                   │
│                                                                  │
│  ZooKeeper / KRaft (metadata, leader election)                  │
└─────────────────────────────────────────────────────────────────┘
```

### Core Concepts

**Topics & Partitions:**
- A **topic** is a named category for messages (like a table in a database, but for streams)
- Topics are split into **partitions** — ordered, immutable sequences of records
- Each partition is replicated across N brokers (configurable replication factor)
- One broker is the **leader** for each partition; others are **followers** (replicas)
- More partitions = higher parallelism, but higher metadata overhead

**The Log:**
- Each partition is a **commit log** — new messages always appended at the end
- Messages are identified by their **offset** (monotonically increasing integer)
- Messages are **retained** for a configurable period (default: 7 days) or size limit — regardless of whether they were consumed
- This is the defining feature: **messages aren't deleted after consumption**

**Producers:**
- Producers decide which partition to write to:
  - **Round-robin** (default for null keys)
  - **Key-based hashing**: Same key always goes to the same partition (ordering guarantee per key)
  - **Custom partitioner**

**Consumer Groups:**
- Consumers organize into **consumer groups** identified by a `group.id`
- Each partition is assigned to exactly one consumer within a group
- Multiple groups can independently consume the same topic — each group has its own offset cursor
- **Rebalancing**: When consumers join/leave a group, partitions are redistributed

```
Topic: "orders" (3 partitions)

Consumer Group A (payment-service):
  Consumer A1 → Partition 0
  Consumer A2 → Partition 1
  Consumer A3 → Partition 2

Consumer Group B (analytics-service):
  Consumer B1 → Partition 0, 1   (fewer consumers than partitions)
  Consumer B2 → Partition 2
```

**Offsets & Commit:**
- Each consumer tracks its position as an **offset** per partition
- Offsets are committed to a special internal Kafka topic (`__consumer_offsets`)
- Consumers can **seek** to any offset — beginning of time, specific timestamp, specific offset
- This enables **replay**: re-read historical messages at any time

### KRaft Mode (Post-ZooKeeper)

Kafka historically required **Apache ZooKeeper** for metadata management and leader election. Kafka 2.8+ introduced **KRaft** (Kafka Raft Metadata mode), making Kafka fully self-managed without ZooKeeper. KRaft is production-ready from Kafka 3.3+ and is now the recommended deployment model, simplifying operations significantly.

### Kafka Streams & ksqlDB

Kafka's ecosystem extends beyond simple pub/sub:
- **Kafka Streams**: Client library for stream processing directly on Kafka data (joins, aggregations, windowing) — no separate cluster needed
- **ksqlDB**: SQL-like streaming query language on top of Kafka
- **Kafka Connect**: Plugin framework for integrating Kafka with databases, S3, Elasticsearch, etc. (source and sink connectors)
- **Schema Registry** (Confluent): Enforces Avro/Protobuf/JSON Schema on topics

### Key Features

- **Horizontal scalability**: Add brokers and partition count independently
- **Durability**: Replication factor N means N-1 broker failures are tolerated
- **Exactly-once semantics (EOS)**: Available since Kafka 0.11 via idempotent producers and transactional APIs
- **High throughput**: Batching, compression (gzip, snappy, lz4, zstd), and zero-copy I/O via `sendfile()` syscall
- **Log compaction**: Topics can be configured for compaction — only the latest value per key is retained (useful as a changelog/state store)
- **Tiered storage** (Kafka 3.6+): Cold data offloaded to S3/GCS while hot data stays on local disk

### Kafka Use Cases

✅ **Great fit for:**
- **Event sourcing**: Kafka as the system of record; services reconstruct state by replaying events
- **Activity tracking**: LinkedIn-style clickstreams, user behavior, audit logs at massive scale
- **Log aggregation**: Centralize application logs from hundreds of services
- **Stream processing pipelines**: ETL, real-time analytics, fraud detection
- **Change Data Capture (CDC)**: Database changelog streamed via Debezium → Kafka
- **Microservices choreography**: Services communicate via domain events on Kafka topics
- **Data integration hub**: One topic, many consumers (analytics, ML, data warehouse, cache)

### Kafka Performance Profile

- **Latency**: Typically 5–15ms end-to-end (tunable; can go lower with `linger.ms=0`)
- **Throughput**: **Millions of messages per second** on a single broker; near-linear scaling with partitions/brokers
- **Message retention**: Days to forever (storage is cheap; Kafka embraces this)
- **Batch efficiency**: Kafka is designed to batch messages — higher throughput, slightly higher latency vs. RabbitMQ

---

## RabbitMQ vs. Kafka: Head-to-Head

### Conceptual Comparison

| Dimension | RabbitMQ | Apache Kafka |
|---|---|---|
| **Paradigm** | Message Queue / Smart Broker | Distributed Log / Event Stream |
| **Message lifecycle** | Deleted after ACK | Retained for configured period |
| **Consumer model** | Broker pushes to consumers | Consumers pull from broker |
| **Routing** | Rich (exchange types, bindings) | Topic + partition only |
| **Ordering** | Per-queue FIFO | Per-partition ordering |
| **Replay** | ❌ Not possible after ACK | ✅ Yes — seek to any offset |
| **Throughput** | Tens of thousands/sec | Millions/sec |
| **Latency** | Sub-ms to low-ms | ~5-15ms typical |
| **Consumer tracking** | Broker manages | Consumer manages offset |
| **Protocol** | AMQP, MQTT, STOMP | Custom binary (Kafka protocol) |
| **Operational complexity** | Moderate | Higher (partitioning strategy matters) |
| **Primary language** | Erlang | Scala/Java |
| **Hosted options** | CloudAMQP, AWS AmazonMQ | Confluent Cloud, AWS MSK, Aiven |

### Performance Benchmarks (Approximate)

```
Throughput (single node, 1KB messages):

RabbitMQ:  ████████░░░░░░░░░░░░  ~50K-100K msg/s
Kafka:     ████████████████████  ~1M-2M+ msg/s

Latency (p99, single node):

RabbitMQ:  [<1ms ████████████]  Excellent for low-latency
Kafka:     [~10ms ████████    ]  Good, but batch-oriented
```

*Note: Benchmarks vary enormously based on hardware, configuration, message size, replication settings, and workload patterns. Always benchmark your specific use case.*

### Ordering Guarantees

**RabbitMQ:**
- Messages in a single queue are FIFO
- With multiple consumers on a queue, ordering is lost at the consumer level
- Priority queues can reorder intentionally

**Kafka:**
- Strict ordering **within a partition**
- No global ordering across partitions in a topic
- Design: use the same partition key for messages that must be ordered together
  ```
  order_id → same partition → ordered processing
  ```

### Message Replay

This is one of the most fundamental differences:

```
RabbitMQ: Consume → ACK → Message gone ✗ (no replay)

Kafka:    Consume → Commit offset (message stays)
          
          Later:   Seek to offset 0 → Replay everything ✓
                   Seek to timestamp → Replay from point in time ✓
                   New consumer group → Starts from beginning ✓
```

**Replay use cases**: Reprocessing after a bug fix, bootstrapping new services, A/B testing different processing logic against the same event history, disaster recovery.

### When to Choose RabbitMQ

- ✅ You need **complex routing** (different message types to different queues)
- ✅ You need **low latency** (sub-millisecond delivery)
- ✅ Tasks that should be **processed once and discarded**
- ✅ **RPC / request-reply** patterns
- ✅ **Per-message TTL** and priority are important
- ✅ You have **heterogeneous protocols** (MQTT for IoT, AMQP for services, STOMP for web)
- ✅ Team is unfamiliar with distributed systems; simpler ops model needed
- ✅ Message volumes are moderate (< 100K/sec)
- ✅ You need **fine-grained message control** (reject, requeue, nack)

**Example scenarios**: Job scheduler dispatching tasks to workers, email/SMS notification service, order fulfillment workflows, IoT data ingestion at moderate scale.

### When to Choose Kafka

- ✅ You need **event replay / reprocessing**
- ✅ **Very high throughput** (millions of events/sec)
- ✅ **Multiple independent consumers** reading the same stream
- ✅ Building an **event sourcing** architecture
- ✅ **Stream processing** with Kafka Streams or Flink
- ✅ **Audit log** or compliance requirement to retain all events
- ✅ **Change Data Capture (CDC)** pipelines
- ✅ **Decoupling microservices** via domain events at scale
- ✅ Data integration hub feeding analytics, ML, and operational systems simultaneously

**Example scenarios**: Real-time analytics pipeline, microservices event bus at scale, fraud detection system, GDPR audit trail, machine learning feature pipeline.

---

## Other Notable Brokers

### Amazon SQS & SNS

**Amazon Simple Queue Service (SQS)** is AWS's fully managed message queue service, available since 2004 — one of the oldest AWS services.

**SQS Queue Types:**
| Feature | Standard Queue | FIFO Queue |
|---|---|---|
| Ordering | Best-effort | Strict FIFO |
| Throughput | Nearly unlimited | 3,000 msg/s (with batching) |
| Delivery | At-least-once | Exactly-once processing |
| Deduplication | ❌ | ✅ (5-min dedup window) |

**Key SQS features:**
- **Visibility timeout**: Message becomes invisible to other consumers while being processed; if not deleted within timeout, becomes visible again (fault tolerance)
- **Long polling**: Consumers wait up to 20 seconds for messages (reduces empty receive calls)
- **Dead Letter Queue**: After N failed deliveries, move to DLQ
- **Message retention**: 1 minute to 14 days
- **Max message size**: 256 KB (larger payloads: store in S3, send S3 reference)

**Amazon Simple Notification Service (SNS)** is a pub/sub service for fan-out patterns. SNS topics fan out to: SQS queues, Lambda functions, HTTP endpoints, email, SMS, mobile push.

**The SQS+SNS Pattern (Fan-out):**
```
Publisher → SNS Topic → SQS Queue A → Consumer A (payment)
                      → SQS Queue B → Consumer B (inventory)
                      → SQS Queue C → Consumer C (notifications)
                      → Lambda      → Consumer D (analytics)
```

✅ **Choose SQS/SNS when**: You're AWS-native, want zero ops overhead, need simple queue or fan-out, moderate throughput, no replay needed.

---

### NATS

**NATS** (Neural Autonomic Transport System) is an ultra-lightweight, high-performance messaging system created by Derek Collison at Apcera (2011), now governed by the NATS.io foundation.

**Core characteristics:**
- Written in **Go**; server binary is ~20MB
- **Fire-and-forget** at its core: if no subscriber is listening, the message is dropped (at-most-once by default)
- Incredibly **low latency**: sub-millisecond, even at high throughput
- Simple **text-based protocol** over TCP; easy to debug with raw `telnet`
- **Subject-based addressing**: Hierarchical subjects with wildcard subscriptions (`orders.>`, `payments.*`)
- **NATS JetStream** (added 2021): Persistence, delivery guarantees (at-least-once, exactly-once), consumer groups, key-value store, and object store — making NATS competitive with Kafka for many use cases

**NATS modes:**
- **Core NATS**: Pub/Sub, Request/Reply, Queue Groups — in-memory, no persistence
- **JetStream**: Streams (persistent, replayable) and Consumers (push or pull-based)

✅ **Choose NATS when**: You need extremely low latency, edge/IoT messaging, service mesh communication, or a lightweight Kafka alternative with JetStream.

---

### Apache Pulsar

**Apache Pulsar** was created at **Yahoo!** in 2013 and open-sourced in 2016, graduating as a top-level Apache project in 2018. It's often positioned as "Kafka done right" or "the next generation Kafka."

**Architecture — Segmented Storage:**
```
┌──────────────────────────────────────────────────────────┐
│                     Apache Pulsar                         │
│                                                           │
│  Broker Layer (stateless):                               │
│  [Broker 1] [Broker 2] [Broker 3]                        │
│                  ↓                                        │
│  Storage Layer (Apache BookKeeper):                       │
│  [Bookie 1] [Bookie 2] [Bookie 3] [Bookie 4]            │
│                  ↓                                        │
│  Metadata (ZooKeeper / etcd)                             │
└──────────────────────────────────────────────────────────┘
```

**Key innovations over Kafka:**
- **Separation of compute and storage**: Brokers are stateless; storage is delegated to **Apache BookKeeper**. This means you can scale brokers and storage independently.
- **Multi-tenancy built-in**: Namespaces and tenants are first-class concepts with per-tenant quotas and isolation.
- **Tiered storage**: Native integration with S3/GCS/HDFS for cold storage.
- **Geo-replication**: Built-in, configurable replication across data centers.
- **Both queue and stream semantics**: Native support for competing consumers (like RabbitMQ) AND exclusive/shared/key-shared subscriptions.
- **Schema registry built-in**: No need for separate Confluent Schema Registry.
- **Functions**: Lightweight serverless compute (Pulsar Functions) similar to Kafka Streams but simpler.

✅ **Choose Pulsar when**: You need multi-tenancy, geo-replication, independent scaling of compute/storage, or want Kafka-like semantics with more operational flexibility.

---

### Apache ActiveMQ

**Apache ActiveMQ** is one of the oldest and most battle-tested open-source message brokers, started in 2004. It's the predecessor to many modern brokers and implements multiple standards:

- **JMS (Java Message Service)**: The Java standard API for messaging
- **AMQP, MQTT, STOMP, OpenWire**: Multiple protocol support
- **Virtual Destinations**: Topic-to-queue bridging

**Two active versions:**
- **ActiveMQ Classic (5.x)**: The original, widely deployed, JMS-focused
- **ActiveMQ Artemis**: Next-generation engine with better performance; this is also the foundation for **Red Hat AMQ**

**Strengths**: Java ecosystem integration, JMS compliance, mature tooling, easy to embed in Java applications.

**Weaknesses**: Lower throughput than Kafka/Pulsar, less cloud-native, slower pace of innovation.

✅ **Choose ActiveMQ when**: You're in a Java/JEE environment, need JMS compliance, integrating with legacy enterprise systems, or using Red Hat middleware stack.

---

## Decision Framework

### The 5-Question Decision Tree

```
1. Do you need message replay / reprocessing?
   YES → Kafka or Pulsar
   NO  → Continue ↓

2. Do you need > 500K messages/second?
   YES → Kafka or Pulsar
   NO  → Continue ↓

3. Do you need complex routing (per-message rules)?
   YES → RabbitMQ
   NO  → Continue ↓

4. Are you fully AWS-native and want zero ops?
   YES → SQS/SNS
   NO  → Continue ↓

5. Do you need ultra-low latency (<1ms) or edge/IoT?
   YES → NATS
   NO  → RabbitMQ (versatile default)
```

### Requirement Matrix

| Requirement | RabbitMQ | Kafka | SQS/SNS | NATS | Pulsar |
|---|---|---|---|---|---|
| Complex routing | ⭐⭐⭐ | ⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| High throughput | ⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Low latency | ⭐⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐ | ⭐⭐ |
| Message replay | ❌ | ⭐⭐⭐ | ❌ | ⭐⭐ (JS) | ⭐⭐⭐ |
| Multi-tenancy | ⭐ | ⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐⭐ |
| Geo-replication | ⭐⭐ | ⭐⭐ | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| Managed cloud | ⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ |
| Ops simplicity | ⭐⭐⭐ | ⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
| Ecosystem | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ | ⭐⭐ | ⭐⭐ |
| Java/JMS | ⭐⭐ | ⭐⭐ | ⭐ | ⭐ | ⭐⭐ |

*⭐⭐⭐ = Excellent, ⭐⭐ = Good, ⭐ = Basic, ❌ = Not supported*

### Anti-Patterns to Avoid

❌ **Using Kafka as a task queue**: Kafka doesn't delete messages after consumption. If you use Kafka for traditional job queues, you either waste storage or need complex offset management. Use RabbitMQ or SQS for this.

❌ **Using RabbitMQ for event sourcing**: Once a message is ACKed, it's gone. You can't replay history. If you realize you had a processing bug, you've lost the events.

❌ **Too many Kafka partitions**: More partitions = more parallelism, BUT more file handles, more ZooKeeper/KRaft load, longer failover times, and rebalancing overhead. Start conservative, scale up.

❌ **Not setting a Dead Letter Queue**: Failed messages that have nowhere to go will block your queue (RabbitMQ) or be silently skipped (if not handled in consumer code).

❌ **Polling SQS without long polling**: Short polling (default) makes many empty API calls. Enable long polling (`WaitTimeSeconds=20`) to reduce cost and empty receives.

❌ **Ignoring consumer group rebalancing in Kafka**: Frequent rebalances (due to slow consumers or rolling deployments) cause processing pauses. Use static membership (`group.instance.id`) for long-lived consumers.

---

## Common Patterns & Anti-Patterns

### Pattern: Competing Consumers (Work Queue)

```
Producer → [Queue] → Consumer 1 (processing)
                   → Consumer 2 (processing)
                   → Consumer 3 (processing)

Used for: Horizontal scaling of job processing
Broker: RabbitMQ, SQS, NATS Queue Groups, Kafka (consumers in same group)
```

### Pattern: Publish-Subscribe (Fan-Out)

```
Producer → Topic/Exchange → Consumer A (payments)
                          → Consumer B (inventory)
                          → Consumer C (notifications)

Used for: Broadcasting events to multiple independent consumers
Broker: RabbitMQ (fanout exchange), SNS, Kafka (multiple consumer groups)
```

### Pattern: Event Sourcing with Kafka

```
Commands → [Command Handler] → Events → Kafka Topic
                                              ↓
                             State rebuilt by replaying events
                             Multiple read models from same events
                             Time-travel debugging
```

### Pattern: Saga (Distributed Transaction)

```
Order Service → "order.placed" → Kafka/RabbitMQ
                                      ↓
Payment Service consumes → processes → publishes "payment.confirmed" or "payment.failed"
                                      ↓
Inventory Service consumes → processes → publishes "inventory.reserved" or "inventory.failed"
                                      ↓
Shipping Service consumes → "order.fulfilled"

On failure: Compensating transactions published as events
```

### Pattern: Outbox Pattern (Transactional Messaging)

Solves the dual-write problem: saving to DB and publishing to broker atomically.
```
1. Write to DB + write to "outbox" table in SAME transaction
2. Separate outbox processor reads new outbox rows
3. Outbox processor publishes to message broker
4. On success, mark outbox row as published

Result: DB and broker are always consistent
Tools: Debezium (CDC-based outbox), Transactional Outbox libraries
```

---

## Summary Reference Card

```
┌─────────────────────────────────────────────────────────────────┐
│                    QUICK REFERENCE CARD                          │
├──────────────┬─────────────────────────────────────────────────┤
│ RabbitMQ     │ Complex routing, low latency, task queues,       │
│              │ RPC, moderate throughput, AMQP ecosystem         │
├──────────────┼─────────────────────────────────────────────────┤
│ Apache Kafka │ Event streaming, replay, high throughput,        │
│              │ CDC, event sourcing, multiple consumers          │
├──────────────┼─────────────────────────────────────────────────┤
│ Amazon SQS   │ AWS-native, zero-ops queuing, simple fan-out     │
│ + SNS        │ with SNS, serverless-friendly                    │
├──────────────┼─────────────────────────────────────────────────┤
│ NATS         │ Ultra-low latency, edge/IoT, lightweight,        │
│              │ service mesh, JetStream for persistence          │
├──────────────┼─────────────────────────────────────────────────┤
│ Apache Pulsar│ Multi-tenant, geo-replication, stateless         │
│              │ brokers, both queue+stream semantics             │
├──────────────┼─────────────────────────────────────────────────┤
│ ActiveMQ     │ Java/JEE, JMS compliance, legacy enterprise,     │
│              │ embedded use cases                               │
└──────────────┴─────────────────────────────────────────────────┘

Golden Rule:
  "Use RabbitMQ when you want to deliver messages.
   Use Kafka when you want to store and replay events."
```

---

## Further Reading

- [RabbitMQ Official Docs](https://www.rabbitmq.com/documentation.html)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Confluent's Kafka tutorials](https://developer.confluent.io/)
- [NATS Documentation](https://docs.nats.io/)
- [Apache Pulsar Documentation](https://pulsar.apache.org/docs/)
- *Designing Data-Intensive Applications* — Martin Kleppmann (Chapter 11: Stream Processing)
- *Enterprise Integration Patterns* — Gregor Hohpe & Bobby Woolf
- [AWS SQS Best Practices](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-best-practices.html)

---

*This article is part of the System Design Wiki — Infrastructure Tools section.*
