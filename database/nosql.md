---
title: "NoSQL"
category: database
summary: "NoSQL databases are designed for horizontal scalability and high availability, trading ACID guarantees for partition tolerance and relaxed consistency models while supporting unnormalized data storage."
sources:
  - raw/articles/_done/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:24:57.510Z
---

# NoSQL

> NoSQL databases are designed for horizontal scalability and high availability, trading ACID guarantees for partition tolerance and relaxed consistency models while supporting unnormalized data storage.

# NoSQL

NoSQL databases emerged in the 2000s as large tech companies needed data storage solutions designed for high availability and scalability. Unlike traditional relational databases built for single-machine deployment, NoSQL systems are natively designed with [[Database Sharding]] in mind.

## Design Philosophy

NoSQL databases make fundamental trade-offs compared to SQL databases:
- **Consistency**: Relax ACID guarantees in favor of eventual/causal consistency
- **Schema**: No strict schema enforcement, enabling flexible data models
- **Joins**: Limited or no join support, relying on unnormalized data storage
- **Transactions**: Limited transaction support due to distributed nature

## Historical Context

Foundational papers like Dynamo and Bigtable established the theoretical basis for modern NoSQL systems. Early implementations were inefficient compared to SQL databases, but modern systems like HBase and Cassandra have achieved high performance.

Many NoSQL databases now support SQL-like syntax despite the "No SQL" name.

## Data Models

**Key-Value Stores**: Simple key-value pair storage
**Document Stores**: Enable indexing on internal document structure

Both models avoid strict schemas and support flexible data representation.

## DynamoDB Example

DynamoDB demonstrates NoSQL design principles:
- **Partition Key**: Determines data distribution across nodes
- **Sort Key (Clustering Key)**: Enables efficient range queries within partitions
- **Replication**: Three replicas per partition using state machine replication
- **Consistency Options**: Eventually consistent (any replica) or strongly consistent (leader only)

### API Operations
- CRUD operations on single items
- Queries with same partition key and sort key filtering
- Full table scans
- **No joins by design**

### Data Modeling

Requires **access pattern-driven design**:
- Model tables based on known query patterns
- Use customer ID as partition key, order time as sort key for customer order queries
- Store multiple entity types in same table using same partition key
- Leverage secondary indexes for complex access patterns

## Key Considerations

**Advantages**:
- Horizontal scalability from day one
- High availability and partition tolerance
- Handles most relational use cases when properly designed

**Disadvantages**:
- Requires upfront access pattern knowledge
- Less flexible than SQL for changing query patterns
- Anti-pattern: Storing relational models in NoSQL results in worst of both worlds

**Recommendation**: Study "The DynamoDB Book" for comprehensive NoSQL design patterns, applicable to other NoSQL databases.

NoSQL databases enable [[System Scaling]] through native distribution while requiring careful data modeling aligned with application access patterns.

---
*Related: [[Database Sharding]], [[System Scaling]], [[NewSQL]], [[Partitioning Strategies]], [[CAP Theorem]]*
