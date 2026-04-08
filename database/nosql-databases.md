---
title: "NoSQL Databases"
category: database
summary: "NoSQL databases are designed for horizontal scalability and high availability, trading ACID guarantees for partition tolerance. They store data as key-value pairs or documents without strict schemas."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:48:15.249Z
---

# NoSQL Databases

> NoSQL databases are designed for horizontal scalability and high availability, trading ACID guarantees for partition tolerance. They store data as key-value pairs or documents without strict schemas.

# NoSQL Databases

NoSQL databases are designed for horizontal scalability and high availability, trading ACID guarantees for partition tolerance. They store data as key-value pairs or documents without strict schemas.

## Design Philosophy

Traditional relational databases were designed assuming single-machine deployment, supporting features like [[ACID Properties]] and joins that become challenging at scale. NoSQL databases were built from the ground up for distributed environments.

Key differences:
- Storage is cheap, CPU time isn't (opposite of historical assumptions)
- Favor denormalized data over joins
- Eventual consistency over strong consistency
- Horizontal scaling over vertical scaling

## NoSQL Flavors

**Key-Value Stores**: Simple key-value pairs with basic CRUD operations

**Document Stores**: Store structured documents (JSON/XML) with indexing on internal structure

Both typically avoid strict schemas, allowing flexible data models.

## Foundational Papers

The Dynamo and Bigtable papers were foundational for modern NoSQL systems. Modern databases like HBase and Cassandra are based on these designs.

## DynamoDB Example

DynamoDB illustrates NoSQL design principles:
- Tables contain items with partition keys and optional sort keys
- Partition key determines data distribution
- Sort key enables efficient range queries within partitions
- Three replicas per partition using state machine replication
- Writes go to leader, acknowledged when 2/3 replicas confirm
- Reads can be eventually consistent (any replica) or strongly consistent (leader only)

## Data Modeling

NoSQL requires modeling data around access patterns:
- Design tables based on query patterns, not entities
- Multiple entity types can share the same table
- Use secondary indexes for additional access patterns
- Local secondary indexes: different sort keys, same partition key
- Global secondary indexes: different partition and sort keys (eventually consistent)

## Trade-offs

**Advantages**: Horizontal scalability, high availability, flexible schemas

**Disadvantages**: Limited query flexibility, requires upfront access pattern knowledge, no joins

The main prerequisite is knowing access patterns beforehand since NoSQL databases are difficult to alter later.

---
*Related: [[NewSQL Databases]], [[Database Sharding]], [[ACID Properties]], [[Dynamo-Style Data Stores]]*
