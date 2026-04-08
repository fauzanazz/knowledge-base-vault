---
title: "ETL/ELT Pipeline Patterns"
category: automated-workflow
summary: "A comprehensive guide to ETL and ELT pipeline patterns covering paradigm differences, batch/streaming processing, modern data stack tools, orchestration, Change Data Capture, schema evolution, data quality, and idempotent pipeline design."
sources:
  - web-research-2026
updated: 2026-04-08T18:30:00.000Z
---

# ETL/ELT Pipeline Patterns

> A comprehensive guide to ETL and ELT pipeline patterns covering paradigm differences, batch/streaming processing, modern data stack tools, orchestration, Change Data Capture, schema evolution, data quality, and idempotent pipeline design.

---

## ETL vs ELT: The Paradigm Shift

### Traditional ETL (Extract → Transform → Load)
In classic ETL, data is extracted from source systems, transformed in a dedicated processing layer (staging server or ETL tool), and then loaded into the target warehouse in a pre-shaped format.

**When to use ETL:**
- Sensitive/regulated data needing masking before landing in the warehouse
- Legacy warehouses with limited compute (e.g., older on-prem SQL Server)
- Strict schema requirements at ingest time
- Small-to-medium data volumes where transformation overhead is acceptable

### Modern ELT (Extract → Load → Transform)
ELT loads raw data into the target system first, then leverages the warehouse's native compute for transformation. Cloud warehouses (Snowflake, BigQuery, Redshift) make this economical because storage is cheap and compute scales elastically.

**When to use ELT:**
- Cloud-native data warehouse environments
- Need for raw data preservation and reprocessing flexibility
- dbt-driven transformation workflows
- Large-scale analytics workloads where warehouse compute outperforms ETL servers

### Key Differences at a Glance

| Dimension | ETL | ELT |
|---|---|---|
| Transform location | External processing layer | Inside the warehouse |
| Raw data retention | Often discarded | Preserved in raw layer |
| Latency | Higher (more hops) | Lower (load first, transform later) |
| Scalability | Limited by ETL server | Scales with warehouse |
| Tooling | Informatica, SSIS, Talend | dbt + Airbyte/Fivetran/Stitch |
| Cost profile | Compute-heavy ETL servers | Storage + warehouse compute |
| Reprocessing | Requires re-extract | Re-run transformations on raw data |

---

## Processing Paradigms

### Batch Processing
Data is collected over a period and processed in bulk at scheduled intervals (hourly, daily, weekly).

**Characteristics:**
- High throughput, lower operational complexity
- Suitable for reporting, analytics, and historical loads
- Tools: Apache Spark, dbt (scheduled runs), Airflow DAGs

**Patterns:**
- **Full refresh**: Truncate and reload entire dataset (safe but expensive)
- **Incremental append**: Load only new records since last run using watermark columns (`created_at`, `updated_at`)
- **Merge/upsert**: Identify changed records via key + timestamp and merge into target

### Streaming Processing
Data is processed continuously as it arrives, enabling near-real-time analytics.

**Characteristics:**
- Sub-second to second latency
- Higher operational complexity (stateful processing, exactly-once semantics)
- Tools: Apache Kafka + Kafka Streams, Apache Flink, Spark Structured Streaming, Materialize

**Patterns:**
- **Event sourcing**: All state changes stored as immutable events in a log (Kafka topics)
- **Windowed aggregations**: Tumbling, sliding, or session windows over event streams
- **Stream-table joins**: Enrich live events with dimension data from tables

### Micro-Batch Processing
A hybrid approach: data is buffered for short windows (seconds to minutes) then processed in small batches. Balances latency and complexity.

**Tools:** Spark Structured Streaming (trigger intervals), AWS Kinesis Data Analytics, Flink with checkpointing

**Trade-off:** Achieves near-real-time at lower complexity than pure streaming, but not true event-by-event processing.

---

## The Modern Data Stack

### Ingestion Layer

| Tool | Type | Best For |
|---|---|---|
| **Airbyte** | Open-source ELT connector | Self-hosted, 300+ connectors, custom connector SDK |
| **Fivetran** | Managed ELT connector | Zero-maintenance, automatic schema migration |
| **Stitch** | Managed ELT (lightweight) | Simpler use cases, Singer protocol-based |
| **Meltano** | Open-source (Singer-based) | GitOps-friendly, developer-centric |
| **Kafka Connect** | Streaming ingest | Event-driven sources (CDC, IoT, logs) |

### Transformation Layer

**dbt (data build tool)** is the de facto standard for ELT transformations:
- SQL-first transformations compiled and run inside the warehouse
- Dependency graph (DAG) between models via `ref()` function
- Built-in testing (`not_null`, `unique`, `accepted_values`, custom tests)
- Documentation and lineage auto-generated
- Incremental models with `is_incremental()` flag

```sql
-- dbt incremental model example
{{ config(materialized='incremental', unique_key='event_id') }}

SELECT
  event_id,
  user_id,
  event_type,
  occurred_at
FROM {{ source('raw', 'events') }}

{% if is_incremental() %}
  WHERE occurred_at > (SELECT MAX(occurred_at) FROM {{ this }})
{% endif %}
```

### Storage Layer

**Data Warehouses:** Snowflake, Google BigQuery, Amazon Redshift, Databricks SQL

**Data Lakes / Lakehouses:**
- **Delta Lake**: ACID transactions on object storage (S3/ADLS), time travel, schema enforcement. Native to Databricks.
- **Apache Iceberg**: Open table format with hidden partitioning, snapshot isolation, row-level deletes. Supported by Spark, Flink, Trino, Athena.
- **Apache Hudi**: Upsert-optimized format for near-real-time ingestion on lakes.

**Lakehouse advantages over pure lake:**
- ACID compliance (no corrupted partial writes)
- Schema enforcement + evolution
- Time travel / data versioning
- Unified batch + streaming reads

---

## Orchestration

Pipeline orchestration manages scheduling, dependency resolution, retries, alerting, and observability.

| Tool | Model | Key Strengths |
|---|---|---|
| **Apache Airflow** | DAG-based (Python) | Mature ecosystem, huge community, operator library |
| **Dagster** | Asset-based (software-defined assets) | Data-aware scheduling, rich UI, type-checked I/O |
| **Prefect** | Flow-based (Python) | Dynamic workflows, hybrid execution, simpler dev experience |
| **Mage** | Block-based notebook-style | Rapid iteration, built-in testing, ELT-native |
| **dbt Cloud** | Job scheduling for dbt | Seamless dbt integration, CI/CD for transforms |

### Airflow DAG Pattern (Batch ELT)
```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

with DAG(
    dag_id="elt_pipeline",
    schedule_interval="@daily",
    start_date=datetime(2026, 1, 1),
    catchup=True,          # enables backfill
    max_active_runs=3,
    default_args={"retries": 2, "retry_delay": timedelta(minutes=5)},
) as dag:
    extract = BashOperator(task_id="extract", bash_command="airbyte sync ...")
    transform = BashOperator(task_id="transform", bash_command="dbt run --select staging+")
    test = BashOperator(task_id="test", bash_command="dbt test --select staging+")
    extract >> transform >> test
```

---

## Change Data Capture (CDC)

CDC captures row-level changes (INSERT/UPDATE/DELETE) from source databases in near-real-time by reading the database transaction log (binlog/WAL/redo log).

### Debezium
The most widely adopted open-source CDC platform. Supports MySQL, PostgreSQL, MongoDB, Oracle, SQL Server, and more.

**Architecture:**
```
Source DB → Debezium Connector → Kafka Topic → Sink Connector → Warehouse/Lake
```

**Event envelope format:**
```json
{
  "op": "u",              // u=update, c=create, d=delete, r=read (snapshot)
  "before": { "id": 1, "status": "pending" },
  "after":  { "id": 1, "status": "shipped" },
  "source": { "ts_ms": 1712599200000, "table": "orders" }
}
```

**CDC vs Poll-based ingestion:**

| | CDC | Poll-based (Airbyte/Fivetran) |
|---|---|---|
| Latency | Milliseconds | Minutes to hours |
| Captures deletes | Yes | Only with soft-delete pattern |
| Source load | Low (log read) | Higher (query-based) |
| Complexity | Higher (Kafka, connector mgmt) | Lower (managed SaaS) |
| Cost | Infrastructure cost | SaaS subscription |

---

## Schema Evolution

Schemas change over time. Good pipelines handle evolution without breaking downstream consumers.

### Strategies

1. **Schema Registry** (Confluent Schema Registry / AWS Glue Schema Registry)
   - Enforces compatibility rules: BACKWARD, FORWARD, FULL
   - Prevents producers from publishing breaking changes

2. **Column addition** (non-breaking): Add new nullable columns; existing consumers ignore unknown fields.

3. **Column removal** (breaking): Use deprecation period — keep old column, backfill new column, migrate consumers, then drop.

4. **Type widening**: `INT` → `BIGINT`, `VARCHAR(50)` → `VARCHAR(255)` are generally safe.

5. **dbt schema evolution**: `on_schema_change: append_new_columns` in incremental models automatically handles new source columns.

### Iceberg Schema Evolution
Apache Iceberg tracks schema history by column ID (not name), enabling safe:
- Column renames (ID stays the same)
- Type promotions
- Column reorders

---

## Data Quality

### Great Expectations
Open-source framework for defining, documenting, and validating data expectations.

```python
import great_expectations as gx

context = gx.get_context()
batch = context.sources.pandas_default.read_dataframe(df)

# Define expectations
batch.expect_column_values_to_not_be_null("user_id")
batch.expect_column_values_to_be_between("age", min_value=0, max_value=120)
batch.expect_column_values_to_be_unique("order_id")

# Validate
results = batch.validate()
```

### Soda
Declarative YAML-based data quality checks, integrates with dbt and Airflow.

```yaml
# soda/checks/orders.yml
checks for orders:
  - row_count > 0
  - missing_count(user_id) = 0
  - duplicate_count(order_id) = 0
  - freshness(created_at) < 2h
  - avg(order_total) between 10 and 500
```

### dbt Tests
Native dbt tests run post-transformation:
- **Generic**: `not_null`, `unique`, `accepted_values`, `relationships`
- **Custom**: SQL-based tests returning zero rows on pass
- **dbt-expectations**: Port of Great Expectations syntax for dbt

### Data Quality Gates
Insert quality checks as blocking steps between pipeline stages:
```
Extract → Load Raw → [Quality Gate] → Transform → [Quality Gate] → Publish
```
Fail fast: reject bad data early rather than propagating errors downstream.

---

## Idempotent Pipelines

An idempotent pipeline produces the same result regardless of how many times it is run for a given logical window.

### Why It Matters
- Safe retries on failure
- Clean backfills without duplicates
- Predictable state after any run count

### Patterns

1. **Truncate-and-reload partitions**: For each run, delete the target partition and rewrite it completely.
   ```sql
   DELETE FROM analytics.orders WHERE date_partition = '{{ ds }}';
   INSERT INTO analytics.orders SELECT ... WHERE date(created_at) = '{{ ds }}';
   ```

2. **MERGE/UPSERT**: Use a primary key to insert new rows or update existing ones.
   ```sql
   MERGE INTO target t USING source s ON t.id = s.id
   WHEN MATCHED THEN UPDATE SET ...
   WHEN NOT MATCHED THEN INSERT ...;
   ```

3. **Write-audit-publish pattern**: Write to a staging table, validate, then atomically swap/promote.

4. **Deterministic surrogate keys**: Hash-based keys (`MD5(concat(...))`) ensure the same source data always produces the same key.

---

## Backfill Strategies

Backfilling re-processes historical data, needed after: pipeline bugs, schema changes, new business logic, or data corrections.

### Approaches

| Strategy | Description | Use Case |
|---|---|---|
| **Full backfill** | Reload all historical data from scratch | Schema redesign, complete reprocessing |
| **Partial backfill** | Re-run specific date range | Bug fix affecting known window |
| **Incremental backfill** | Process in chunks to avoid resource spikes | Large historical datasets |
| **Shadow backfill** | Write to parallel table, validate, then swap | Zero-downtime migrations |

### Airflow Catchup
Setting `catchup=True` with `start_date` in the past causes Airflow to backfill all missed intervals automatically. Control parallelism with `max_active_runs`.

### dbt Backfill Pattern
```bash
# Re-run incremental models as full refresh for a backfill
dbt run --full-refresh --select orders_mart
```

---

## Pipeline Architecture Patterns

### Lambda Architecture
Combines batch layer (accurate, high-latency) and speed layer (approximate, low-latency) with a serving layer merging both views.
- **Downside**: Dual code paths to maintain.

### Kappa Architecture
Single streaming pipeline handles both real-time and historical processing by replaying the event log.
- Simpler than Lambda but requires a replayable log (Kafka with long retention).

### Medallion Architecture (Bronze/Silver/Gold)
Popularized by Databricks for lakehouses:
- **Bronze**: Raw ingested data, immutable, append-only
- **Silver**: Cleaned, deduplicated, conformed data
- **Gold**: Business-level aggregates and feature tables

---

## Tool Comparison Summary

| Category | Open Source | Managed/SaaS |
|---|---|---|
| **Ingestion** | Airbyte, Meltano, Kafka Connect | Fivetran, Stitch, Census |
| **Transformation** | dbt Core, Apache Spark | dbt Cloud, Databricks |
| **Orchestration** | Airflow, Dagster, Prefect, Mage | Astronomer, Dagster Cloud, Prefect Cloud |
| **Streaming** | Kafka, Flink, Spark Streaming | Confluent Cloud, AWS Kinesis, Google Dataflow |
| **CDC** | Debezium | Fivetran (CDC mode), Estuary Flow |
| **Table Format** | Delta Lake, Iceberg, Hudi | Snowflake (native), BigQuery |
| **Data Quality** | Great Expectations, Soda Core, dbt tests | Soda Cloud, Monte Carlo, Bigeye |
| **Schema Registry** | Confluent Schema Registry | AWS Glue Schema Registry |

---

## Key Takeaways

1. **Prefer ELT** for cloud warehouses — preserve raw data, leverage warehouse compute, enable reprocessing.
2. **Choose batch vs streaming** based on latency requirements and operational tolerance — micro-batch is often the pragmatic middle ground.
3. **CDC with Debezium** is the gold standard for low-latency, delete-aware replication from operational databases.
4. **dbt is the standard** ELT transformation layer — use incremental models with proper unique keys for efficiency.
5. **Design idempotent pipelines** from the start — makes retries, backfills, and debugging dramatically simpler.
6. **Embed data quality gates** between pipeline stages rather than validating only at the end.
7. **Medallion architecture** (Bronze/Silver/Gold) provides a clean, auditable data flow model for lakehouses.
8. **Schema evolution** requires explicit strategy — Schema Registry for streams, column-ID-based tracking (Iceberg) for lake tables.
