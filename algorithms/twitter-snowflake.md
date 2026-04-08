---
title: "Twitter Snowflake"
category: algorithms
summary: "Twitter Snowflake is a distributed unique ID generation algorithm that creates 64-bit time-ordered identifiers by dividing bits into timestamp, datacenter, machine, and sequence sections."
sources:
  - raw/articles/unique-id-generator-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:49:54.856Z
---

# Twitter Snowflake

> Twitter Snowflake is a distributed unique ID generation algorithm that creates 64-bit time-ordered identifiers by dividing bits into timestamp, datacenter, machine, and sequence sections.

# Twitter Snowflake

Twitter Snowflake is a distributed unique ID generation algorithm that creates 64-bit time-ordered identifiers without requiring coordination between servers.

## Architecture

Snowflake divides 64-bit IDs into four distinct sections:

### Sign Bit (1 bit)
Always set to 0, potentially distinguishing between signed and unsigned numbers.

### Timestamp (41 bits)
Stores milliseconds since a custom epoch (Twitter's default: `1288834974657`, equivalent to Nov 04, 2010, 01:42:54 UTC). This ensures IDs are naturally time-ordered and provides approximately 69 years of timestamp range.

### Datacenter ID (5 bits)
Identifies up to `2^5 = 32` different datacenters, enabling geographic distribution.

### Machine ID (5 bits)
Identifies up to `2^5 = 32` machines within each datacenter, supporting horizontal scaling.

### Sequence Number (12 bits)
Tracks IDs generated on a single machine within the same millisecond, supporting up to `2^12 = 4,096` IDs per millisecond. The sequence resets to 0 every millisecond.

## Advantages

- **Scalability:** Handles 10,000+ IDs per second across multiple servers
- **Time-Ordering:** IDs are naturally sortable by generation time
- **Decentralization:** No single point of failure or coordination required
- **Efficiency:** Fits within 64-bit constraint while maximizing information density

## Implementation Considerations

### Clock Synchronization
Requires synchronized clocks across all servers using [[Network Time Protocol]] (NTP) to maintain proper time ordering.

### Configuration Flexibility
Bit allocation can be adjusted based on specific requirements - more timestamp bits for longer lifespan or more sequence bits for higher throughput.

### High Availability
Each machine generates IDs independently, eliminating coordination overhead and single points of failure.

Snowflake has become the de facto standard for distributed unique ID generation in modern [[Microservices Architecture]] and large-scale systems.

---
*Related: [[Unique ID Generator]], [[Distributed Systems]], [[System Scaling]], [[Microservices Architecture]]*
