---
title: "Time-Series Database"
category: database
summary: "A time-series database is specialized for storing and querying time-stamped data points, optimized for high write throughput and efficient aggregation queries. Popular examples include InfluxDB and Prometheus."
sources:
  - raw/articles/metrics-monitoring-and-alerting-system-system-design-interview-by-alex-xu-pagefy.md
updated: 2026-04-04T09:39:01.482Z
---

# Time-Series Database

> A time-series database is specialized for storing and querying time-stamped data points, optimized for high write throughput and efficient aggregation queries. Popular examples include InfluxDB and Prometheus.

# Time-Series Database

A time-series database is a specialized database optimized for storing and querying time-stamped data points. Unlike general-purpose databases, these systems are designed for the unique characteristics of time-series data.

## Key Characteristics

**Data Structure**: Time-series data consists of:
- Timestamp
- Metric name
- Optional labels/tags
- Numeric value

**Access Patterns**:
- Write-heavy workloads with high ingestion rates
- Infrequent but bursty read patterns
- Range queries over time intervals
- Aggregation operations (sum, average, percentiles)

## Popular Systems

**InfluxDB**: Can handle 250k+ writes per second on 8 cores/32GB RAM. Uses in-memory cache with on-disk storage.

**Prometheus**: Pull-based monitoring system with built-in time-series database and query language (PromQL).

**OpenTSDB**: Distributed system built on Hadoop and HBase.

**Cloud Options**: Amazon Timestream, Google Cloud Monitoring.

## Optimization Techniques

**Compression**: Delta encoding for timestamps and values reduces storage requirements significantly.

**Indexing**: Efficient indexes on metric labels enable fast filtering and grouping operations.

**Down-sampling**: Converting high-resolution data to lower resolution for long-term storage.

**Partitioning**: Data partitioned by time ranges and metric names for parallel processing.

## Query Languages

Time-series databases typically use specialized query languages instead of SQL:

**Flux (InfluxDB)**:
```
from(db:"telegraf")
 |> range(start:-1h)
 |> filter(fn: (r) => r._measurement == "cpu")
 |> mean()
```

**PromQL (Prometheus)**:
```
rate(http_requests_total[5m])
```

These languages are optimized for time-series operations like rate calculations, moving averages, and temporal aggregations that would be complex in SQL.

---
*Related: [[Metrics Monitoring and Alerting System]], [[Data Compression]], [[Database Indexing]], [[Query Optimization]]*
