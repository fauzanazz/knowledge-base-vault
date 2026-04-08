---
title: "Ad Click Event Aggregation"
category: system-design
summary: "A system design for aggregating billions of ad click events per day to support real-time bidding and billing at Facebook/Google scale. It processes 1 billion daily clicks with high accuracy requirements for financial transactions."
sources:
  - raw/articles/ad-click-event-aggregation-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:51:06.806Z
---

# Ad Click Event Aggregation

> A system design for aggregating billions of ad click events per day to support real-time bidding and billing at Facebook/Google scale. It processes 1 billion daily clicks with high accuracy requirements for financial transactions.

# Ad Click Event Aggregation

Ad click event aggregation is a critical system for digital advertising platforms that processes billions of click events daily to support real-time bidding (RTB) and billing operations.

## System Requirements

**Scale**: 1 billion ad clicks per day (10,000 QPS average, 50,000 peak QPS) with 30% year-over-year growth. The system must handle 2 million total ads with high accuracy requirements since data impacts advertiser billing.

**Key Queries**:
- Return click count for ad X in last Y minutes
- Return top 100 most clicked ads in past 1 minute
- Support filtering by IP, user_id, country

## Architecture Design

The system uses a **Kappa architecture** combining streaming and batch processing through a [[Message Queue]] (Kafka) and MapReduce framework:

1. **Raw Data Ingestion**: Ad click events flow through Kafka partitions
2. **Aggregation Service**: MapReduce nodes process events in real-time
3. **Storage**: Both raw data (Cassandra) and aggregated results stored
4. **Query API**: REST endpoints serve dashboard and billing systems

## Data Model

**Raw Events**: `ad_id`, `click_timestamp`, `user_id`, `ip`, `country`

**Aggregated Data**: Uses star schema with pre-computed dimensions for fast filtering. Maintains per-minute aggregations and top-N rankings.

## Key Design Decisions

**Time Handling**: Uses event time over processing time for accuracy, with watermarks to handle delayed events. End-of-day reconciliation ensures correctness.

**Delivery Guarantees**: Implements exactly-once semantics using distributed transactions to prevent duplicate billing.

**Scaling**: Independent scaling of message queues, aggregation nodes, and databases. Handles hotspots through resource allocation and data splitting.

**Fault Tolerance**: Snapshots and consumer offsets enable recovery from node failures without data loss.

The system prioritizes data accuracy over latency since billing discrepancies can cost millions of dollars.

---
*Related: [[Message Queue]], [[Real-time Gaming Leaderboard]], [[Microservices Architecture]], [[Database Sharding]], [[Horizontal Scaling]]*
