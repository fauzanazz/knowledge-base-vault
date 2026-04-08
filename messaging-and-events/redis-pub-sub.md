---
title: "Redis Pub/Sub"
category: system-design
summary: "Redis Pub/Sub is a messaging pattern that enables publishers to send messages to channels without knowing specific subscribers. It provides lightweight, real-time message distribution for applications requiring instant notifications or updates."
sources:
  - raw/articles/nearby-friends-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:38:12.596Z
---

# Redis Pub/Sub

> Redis Pub/Sub is a messaging pattern that enables publishers to send messages to channels without knowing specific subscribers. It provides lightweight, real-time message distribution for applications requiring instant notifications or updates.

# Redis Pub/Sub

Redis Pub/Sub (Publish/Subscribe) is a messaging pattern where publishers send messages to named channels without knowledge of specific subscribers. Subscribers listen to channels and receive messages in real-time, making it ideal for building real-time applications.

## Core Concepts

**Publishers**: Send messages to specific channels using the `PUBLISH` command
**Subscribers**: Listen to channels using `SUBSCRIBE` and receive messages as they arrive
**Channels**: Named message routes that connect publishers and subscribers
**Pattern Matching**: Subscribers can use wildcards to subscribe to multiple channels

## Key Characteristics

**Fire-and-Forget**: Messages are not stored; if no subscribers are listening, messages are lost
**Real-time Delivery**: Messages are delivered immediately to active subscribers
**No Memory Overhead**: Channels consume no memory when inactive
**Scalability**: Can handle millions of channels efficiently

## Scaling Redis Pub/Sub

For high-throughput applications like [[Nearby Friends System]], single Redis instances may not suffice:

**Distributed Clusters**: Multiple Redis servers handle different channel subsets using [[Consistent Hashing]]
**[[Service Discovery]]**: Tools like Zookeeper track which servers host specific channels
**Hash Ring Distribution**: Channels are distributed across servers to balance load

## Use Cases

- **Real-time Notifications**: [[Chat System]] message delivery
- **Location Updates**: Broadcasting user location changes to friends
- **Live Updates**: Stock prices, sports scores, news feeds
- **Cache Invalidation**: Notifying application servers of data changes

## Limitations

**No Persistence**: Messages are lost if no subscribers are active
**No Message Ordering**: No guarantees about message delivery order
**No Acknowledgments**: Publishers don't know if messages were received
**Memory Usage**: Subscriber connections consume memory

## Alternatives

For applications requiring message persistence and delivery guarantees, consider:
- **[[Message Queue]]** systems like Apache Kafka
- **Database-based messaging** with polling
- **WebSocket** connections with custom message routing

Redis Pub/Sub excels in scenarios requiring lightweight, real-time message distribution where occasional message loss is acceptable.

---
*Related: [[Message Queue]], [[WebSocket]], [[Chat System]], [[Nearby Friends System]], [[Real-time Systems]]*
