---
title: "Workflow Engines: Temporal vs Airflow vs n8n"
category: automated-workflow
summary: "A deep technical comparison of three leading workflow engines — Temporal (durable execution for distributed systems), Apache Airflow (DAG-based batch/ETL orchestration), and n8n (visual low-code automation) — covering architecture, scaling, fault tolerance, language support, and decision guidance."
sources:
  - web-research-2026
updated: 2026-04-08T18:30:00.000Z
---

# Workflow Engines: Temporal vs Airflow vs n8n

> A deep technical comparison of three leading workflow engines — Temporal (durable execution for distributed systems), Apache Airflow (DAG-based batch/ETL orchestration), and n8n (visual low-code automation) — covering architecture, scaling, fault tolerance, language support, and decision guidance.

---

## What Problem Does Each Tool Solve?

Before comparing features, understand the distinct problem space each tool inhabits:

| Tool | Core Problem | Audience |
|------|-------------|----------|
| **Temporal** | Reliable, long-running distributed application logic that must survive failures | Backend engineers building mission-critical distributed systems |
| **Apache Airflow** | Scheduling and orchestrating complex batch data pipelines (ETL/ELT) | Data engineers managing recurring data workflows |
| **n8n** | Automating actions and integrations across SaaS apps and APIs with minimal code | Ops teams, developers, and non-technical users connecting business tools |

> **Rule of thumb:** If your workflow *is* your product's core logic → Temporal. If it *processes data on a schedule* → Airflow. If it *glues tools together* → n8n.

---

## 1. Apache Airflow

### What It Is

Apache Airflow (born at Airbnb in 2014, Apache top-level project since 2019) is a Python-based platform for programmatically authoring, scheduling, and monitoring batch workflows. The fundamental abstraction is the **DAG (Directed Acyclic Graph)** — a Python file that defines tasks and their dependencies.

### Architecture

Airflow follows a **master-worker** architecture with modular, separately deployable components:

```
┌────────────────────────────────────────────────────────┐
│                   Airflow Cluster                      │
│                                                        │
│  ┌─────────────┐   ┌────────────┐   ┌──────────────┐  │
│  │  Scheduler  │──▶│  Executor  │──▶│  Task Queue  │  │
│  │ (parses DAGs│   │(CeleryExec,│   │(Redis/RabbitMQ│ │
│  │  triggers   │   │ K8sExec,   │   │  or in-proc) │  │
│  │  task runs) │   │ LocalExec) │   └──────┬───────┘  │
│  └──────┬──────┘   └────────────┘          │          │
│         │                                  ▼          │
│  ┌──────▼──────┐                  ┌────────────────┐  │
│  │  Metadata   │                  │  Worker Nodes  │  │
│  │  Database   │                  │ (execute tasks)│  │
│  │(PostgreSQL/ │                  └────────────────┘  │
│  │  MySQL)     │                                       │
│  └─────────────┘                                       │
│                                                        │
│  ┌─────────────┐   ┌─────────────┐                    │
│  │  Webserver  │   │  Triggerer  │ (async sensors)    │
│  │   (UI)      │   │             │                    │
│  └─────────────┘   └─────────────┘                    │
└────────────────────────────────────────────────────────┘
```

**Key Components:**

- **Scheduler** — Continuously parses the DAG directory, triggers DAG runs based on schedules, and pushes ready tasks to the executor. The heart of Airflow.
- **Executor** — Embedded in the scheduler; determines *how* tasks run. Choices include:
  - `LocalExecutor` — parallel tasks in subprocesses on the scheduler host
  - `CeleryExecutor` — distributed workers via Celery + Redis/RabbitMQ
  - `KubernetesExecutor` — spins up a fresh pod per task
  - `CeleryKubernetesExecutor` — hybrid of both
- **Workers** — Processes/pods that pull tasks from the queue and execute them
- **Metadata Database** — PostgreSQL or MySQL stores all DAG/task state, execution history, and XCom values
- **Webserver** — Provides the Airflow UI for monitoring, triggering, and managing DAGs
- **Triggerer** — Asynchronous service (Airflow 2.2+) that handles deferrable operators without blocking a worker slot
- **DAG Processor** — Optionally isolated service for parsing DAG files (security and performance)

### Programming Model

DAGs are Python files. Every task is a callable wrapped in an **Operator**:

```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.providers.postgres.operators.postgres import PostgresOperator
from datetime import datetime

with DAG(
    dag_id="daily_warehouse_refresh",
    start_date=datetime(2025, 1, 1),
    schedule="@daily",
    catchup=False,
    tags=["etl", "warehouse"],
) as dag:

    extract = BashOperator(
        task_id="extract_from_source",
        bash_command="python /opt/scripts/extract.py",
    )

    transform = PostgresOperator(
        task_id="run_dbt_transform",
        postgres_conn_id="warehouse_prod",
        sql="CALL sp_transform_daily();",
    )

    load = PythonOperator(
        task_id="load_to_data_mart",
        python_callable=load_mart,
    )

    extract >> transform >> load  # dependency chaining
```

The **TaskFlow API** (Airflow 2.0+) simplifies Python-native workflows using decorators:

```python
from airflow.decorators import dag, task

@dag(schedule="@hourly", start_date=datetime(2025, 1, 1))
def etl_pipeline():
    @task
    def extract() -> dict:
        return fetch_records()

    @task
    def transform(data: dict) -> dict:
        return process(data)

    @task
    def load(data: dict):
        write_to_db(data)

    load(transform(extract()))

etl_pipeline()
```

### Language Support

Airflow is **Python-only**. DAGs and custom operators must be written in Python. The Python SDK is mature (20,000+ GitHub stars, 80+ official provider packages covering AWS, GCP, Snowflake, Databricks, dbt, Spark, and more).

### Scaling Model

| Layer | Mechanism |
|-------|-----------|
| **Task parallelism** | CeleryExecutor or KubernetesExecutor; add worker nodes horizontally |
| **DAG parsing** | Separate DAG Processor service; tune `max_dagruns_per_loop_to_schedule` |
| **Scheduler HA** | Multiple schedulers with database row locking (Airflow 2.x) |
| **Worker isolation** | KubernetesExecutor gives each task its own pod + resource limits |

Managed offerings: **AWS MWAA**, **Google Cloud Composer**, **Astronomer** handle infrastructure, HA, and scaling automatically.

### Fault Tolerance

- Per-task **retry policies** with configurable delays and max retries
- **Backfill** — re-run past DAG intervals declaratively
- Task state stored in metadata DB; scheduler rebuilds state on restart
- **SLA misses** trigger alerts; sensors can wait for external conditions
- ⚠️ Weakness: mid-run failures can leave tasks in a "zombie" state; the scheduler loop may not catch restarts cleanly under high load

---

## 2. Temporal

### What It Is

Temporal is a **durable execution platform** — a descendant of Uber's Cadence (created by the same engineers). Rather than scheduling DAGs, Temporal lets you write ordinary code (functions/methods) that are guaranteed to complete even through crashes, deployments, and infrastructure failures. As of 2026, Temporal has raised $300M at a $5B valuation, handles 150,000+ actions/second spikes for customers, and has seen 380% revenue growth YoY driven largely by AI agent workloads.

### Architecture

Temporal uses a **control-plane / compute-plane separation**:

```
┌─────────────────────────────────────────────────────────────┐
│                    Temporal Service (Control Plane)          │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │  Frontend    │  │   History    │  │    Matching      │  │
│  │  Service     │  │   Service    │  │    Service       │  │
│  │ (gRPC API)   │  │ (event log / │  │ (task queues /  │  │
│  │              │  │  state mgmt) │  │  task dispatch) │  │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘  │
│         │                 │                   │             │
│  ┌──────▼─────────────────▼───────────────────▼──────────┐  │
│  │              Persistence Layer                         │  │
│  │         (PostgreSQL / MySQL / Cassandra)               │  │
│  │    + Elasticsearch/OpenSearch (visibility/search)      │  │
│  └────────────────────────────────────────────────────────┘  │
└────────────────────────┬────────────────────────────────────┘
                         │ gRPC (task polling)
              ┌──────────▼──────────┐
              │   Worker Fleet      │  ← YOUR infrastructure
              │  (Compute Plane)    │
              │                     │
              │  ┌───────────────┐  │
              │  │ Workflow Code │  │  ← deterministic functions
              │  │ Activity Code │  │  ← any side effects / I/O
              │  └───────────────┘  │
              └─────────────────────┘
```

**Key Internals:**

- **History Service** — Persists every workflow event (task scheduled, completed, failed, signaled). Sharded into 512–1024 shards; shards are the unit of horizontal scaling. Workflow ID is hashed to a shard.
- **Matching Service** — Manages **Task Queues** (logical lanes for dispatching work to workers). Partitioned under the hood for high throughput.
- **Frontend Service** — gRPC gateway; routes client SDK calls to the right internal services.
- **Worker Service** — Internal (not your workers); manages internal background processing tasks.
- **Workers (yours)** — Long-running processes you deploy. They poll task queues and execute workflow/activity code. Temporal never runs your code — it orchestrates it.

### Programming Model

Three core concepts:

- **Workflow** — A deterministic function encoding long-running business logic. Must be deterministic (no random, no `time.now()`, no direct I/O). Temporal replays the event history to reconstruct state.
- **Activity** — A non-deterministic function that does real work (API calls, DB writes, sending email). Has independent retry policies, heartbeating, and timeouts.
- **Worker** — Your process that registers and executes both.

```python
# workflows.py
from temporalio import workflow
from temporalio.common import RetryPolicy
from datetime import timedelta
import activities

@workflow.defn
class OrderFulfillmentWorkflow:

    @workflow.run
    async def run(self, order_id: str) -> str:
        retry = RetryPolicy(maximum_attempts=5, backoff_coefficient=2.0)

        # Each activity call is durable — if the worker crashes here,
        # Temporal replays history and resumes from this point.
        await workflow.execute_activity(
            activities.validate_order,
            order_id,
            schedule_to_close_timeout=timedelta(minutes=5),
            retry_policy=retry,
        )

        payment_result = await workflow.execute_activity(
            activities.charge_payment,
            order_id,
            schedule_to_close_timeout=timedelta(minutes=10),
            retry_policy=retry,
        )

        # Can wait days/weeks — workflow is suspended, not spinning
        await workflow.sleep(timedelta(hours=24))  # wait for shipping

        await workflow.execute_activity(
            activities.send_confirmation_email,
            order_id,
            schedule_to_close_timeout=timedelta(minutes=2),
        )

        return f"Order {order_id} fulfilled"
```

```python
# worker.py — runs in your infrastructure
from temporalio.client import Client
from temporalio.worker import Worker
import asyncio, workflows, activities

async def main():
    client = await Client.connect("localhost:7233")
    worker = Worker(
        client,
        task_queue="order-fulfillment",
        workflows=[workflows.OrderFulfillmentWorkflow],
        activities=[
            activities.validate_order,
            activities.charge_payment,
            activities.send_confirmation_email,
        ],
    )
    await worker.run()

asyncio.run(main())
```

### Language Support

Temporal offers **official, production-grade SDKs** for:

| Language | Maturity | Notes |
|----------|----------|-------|
| **Go** | ⭐⭐⭐⭐⭐ | Most mature; server is written in Go |
| **Java** | ⭐⭐⭐⭐⭐ | Widely used in enterprise |
| **TypeScript** | ⭐⭐⭐⭐ | Fully production-ready (2024+) |
| **Python** | ⭐⭐⭐⭐ | Fully production-ready (2023+) |
| **.NET** | ⭐⭐⭐ | GA; growing adoption |
| **PHP** | ⭐⭐⭐ | Community-driven |
| **Ruby** | ⭐⭐⭐ | GA as of 2025/2026 |

This polyglot support is a major differentiator — different teams can use different languages against the same Temporal cluster.

### Scaling Model

| Layer | Mechanism |
|-------|-----------|
| **Workers** | Horizontally scale your own worker fleet; add more pollers per task queue |
| **History Service** | Sharded (512–1024 shards); add nodes and rebalance shards |
| **Matching Service** | Task queue partitions scale linearly; transparent to SDK callers |
| **Namespaces** | Logical isolation units; cell-based isolation for multi-tenant deployments |
| **Temporal Cloud** | Managed auto-scaling; handled 150K+ actions/sec spikes in production |

### Fault Tolerance

Temporal's durability model is categorically stronger than other tools:

- **Event Sourcing** — Every state transition is appended to an immutable event log. Worker crash = replay from log, no lost progress.
- **At-least-once execution** — Activities are retried until they succeed (or hit max retries).
- **Heartbeating** — Long-running activities report heartbeats; Temporal detects hangs and reschedules.
- **Workflow versioning** — `workflow.patched()` API enables safe code changes to in-flight workflows.
- **Compensation / Sagas** — Model rollback logic as compensating activities in the workflow code.

---

## 3. n8n

### What It Is

n8n (pronounced "n-eight-n") is an open-source, **fair-code** workflow automation platform. Think Zapier or Make.com, but self-hostable and with a code escape hatch. It excels at integrating SaaS tools, APIs, and internal services through a visual drag-and-drop editor with 400+ built-in nodes. By 2026, n8n has 65,000+ GitHub stars and is one of the most rapidly adopted self-hosted automation tools.

### Architecture

n8n is architecturally lightweight compared to Airflow or Temporal:

```
┌──────────────────────────────────────────────────────┐
│                    n8n Instance                      │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │              n8n Core Process                  │  │
│  │                                                │  │
│  │  ┌─────────────┐   ┌────────────────────────┐ │  │
│  │  │  Workflow   │   │   Node Executor         │ │  │
│  │  │  Engine     │──▶│ (HTTP, DB, code, AI...) │ │  │
│  │  └──────┬──────┘   └────────────────────────┘ │  │
│  │         │                                      │  │
│  │  ┌──────▼──────┐   ┌──────────────────────┐   │  │
│  │  │  Scheduler  │   │   Webhook Listener   │   │  │
│  │  │ (cron-based)│   │  (event triggers)    │   │  │
│  │  └─────────────┘   └──────────────────────┘   │  │
│  └────────────────────────────────────────────────┘  │
│                           │                          │
│  ┌────────────────────────▼─────────────────────┐    │
│  │         Database (SQLite / PostgreSQL)        │    │
│  │    (workflow definitions, execution history,  │    │
│  │     credentials, settings)                    │    │
│  └───────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘

── Queue Mode (horizontal scale) ──────────────────────
  ┌───────────┐    ┌──────────────┐    ┌─────────────┐
  │  n8n Main │    │     Redis    │    │  n8n Worker │
  │ (Webhooks │───▶│   (BullMQ)   │───▶│  Instances  │
  │  /UI)     │    │              │    │ (execution) │
  └───────────┘    └──────────────┘    └─────────────┘
```

**Deployment Modes:**

- **Single instance** — All-in-one: UI, scheduler, webhook listener, executor in one process. SQLite or PostgreSQL backend. Minimum 2GB RAM.
- **Queue mode** — n8n main handles webhooks/UI; Redis (BullMQ) distributes executions; separate n8n worker processes execute workflows. Enables horizontal scaling.
- **n8n Cloud** — Fully managed SaaS; tiered by active workflows and execution count.

### Programming Model

Workflows are built visually in the browser. Each **node** represents a step — a trigger (webhook, schedule, event) or an action (HTTP request, database query, send email). Connections between nodes define data flow.

```
[Webhook Trigger] ──▶ [Set Variables] ──▶ [HTTP Request] ──▶ [If Branch]
                                                                   │
                                              [Slack: Send] ◀──── ┤ (success)
                                                                   │
                                              [Email: Alert] ◀─── ┘ (failure)
```

For custom logic, n8n provides a **Code node** (JavaScript/Python):

```javascript
// Code node: transform and filter incoming data
const items = $input.all();

return items
  .filter(item => item.json.status === 'active')
  .map(item => ({
    json: {
      id: item.json.id,
      email: item.json.email.toLowerCase(),
      fullName: `${item.json.firstName} ${item.json.lastName}`,
      processedAt: new Date().toISOString(),
    }
  }));
```

n8n also supports **sub-workflows**, **error trigger workflows**, and **AI agent nodes** (LangChain integration, tool-calling agents) as of 2025/2026.

### Language Support

n8n workflows are **language-agnostic visually**, but custom code nodes support **JavaScript** (Node.js) and **Python**. Custom nodes (community or private) are written in **TypeScript**. No Python-only or Go SDK — the interface is the visual editor or REST API.

### Scaling Model

| Mode | Mechanism | Use Case |
|------|-----------|----------|
| **Single instance** | Vertical scale (more CPU/RAM) | Dev, low-medium volume |
| **Queue mode** | Redis + multiple worker processes | High-volume, HA production |
| **n8n Cloud** | Managed auto-scale | SaaS teams, fast start |

Queue mode is the key: the main instance handles inbound triggers and the UI; workers pull jobs from Redis and execute them. Workers are stateless and horizontally scalable. Minimum 2GB RAM per instance; typical production setups run 4–8GB.

### Fault Tolerance

- **Node-level retries** — Configure retry count and wait time per node
- **Error trigger workflows** — A dedicated workflow that fires on execution failure, enabling alerting or compensating actions
- **Execution logs** — Full input/output stored per node for debugging
- **Pinned data** — Replay executions with pinned test data
- ⚠️ Weakness: No durable execution guarantee. If a worker crashes mid-execution, that run may be lost (depends on queue mode and Redis persistence). Not suitable for workflows where partial completion has irrecoverable side effects.

---

## Head-to-Head Comparison

| Dimension | Temporal | Apache Airflow | n8n |
|-----------|----------|----------------|-----|
| **Primary use case** | Durable distributed application logic | Batch data pipelines / ETL | SaaS integration & business automation |
| **Abstraction** | Code-first workflows + activities | Python DAGs + operators | Visual nodes + optional code |
| **Language support** | Go, Java, TS, Python, .NET, PHP, Ruby | Python only | Visual (JS/Python code nodes) |
| **Workflow time horizon** | Seconds → years (truly long-running) | Minutes → hours (recurring runs) | Seconds → minutes |
| **State model** | Durable event-sourced history | Metadata DB (task state snapshots) | Transactional (execution records) |
| **Fault tolerance** | ⭐⭐⭐⭐⭐ (guaranteed completion) | ⭐⭐⭐⭐ (retries, backfill) | ⭐⭐⭐ (retries, error workflows) |
| **Scheduling** | Cron + event-driven + signal/await | Cron-based (rich schedule syntax) | Cron + webhook/event triggers |
| **Built-in integrations** | None (you write all connectors) | 80+ provider packages (data tools) | 400+ nodes (SaaS, APIs, DBs) |
| **Scaling model** | Scale workers + History Service shards | Scale Celery workers or K8s pods | Scale queue-mode workers via Redis |
| **Min infrastructure** | 4+ containers (server + DB + workers) | 4+ containers (scheduler, worker, DB, webserver) | 1 container (or 2 with Postgres) |
| **Idle RAM footprint** | ~2 GB (server + DB + UI) | ~3–6 GB (full distributed setup) | ~150–250 MB |
| **Learning curve** | 🔴 Steep (new mental model, determinism rules) | 🟡 Moderate (Python, DAG concepts) | 🟢 Low (visual builder) |
| **UI / observability** | Temporal Web UI + Temporal Cloud UI | Mature DAG/task-level UI | Execution-level visual debugger |
| **Self-hosted** | ✅ (complex setup) | ✅ (complex setup) | ✅ (simple Docker) |
| **Managed cloud** | Temporal Cloud ($$$) | AWS MWAA, Cloud Composer, Astronomer | n8n Cloud (tiered) |
| **Open source license** | MIT | Apache 2.0 | Fair-code (Sustainable Use License) |
| **GitHub stars (2026)** | ~19,000 | ~38,000 | ~65,000 |
| **Best for AI agents** | ✅ (long-running, stateful agents) | ⚠️ (task-based, not agent-native) | ✅ (AI agent nodes, LangChain) |

---

## Architecture Deep-Dive: Key Differences

### State Management Philosophy

```
Temporal:   Workflow state = replayed from immutable event log
            → Guaranteed exactly-once progress, infinite resumability

Airflow:    Workflow state = last-known task status in metadata DB
            → Good for batch reruns, weak for mid-crash recovery

n8n:        Workflow state = execution record (input/output per node)
            → Good for auditing, not for resuming interrupted executions
```

### Execution Trigger Model

```
Temporal:   Client SDK call → Frontend Service → History Service
            → Matching Service task queue → Worker polls and executes
            (event-driven, push-pull with long-polling workers)

Airflow:    Scheduler loop (every N seconds) scans DAGs → checks schedule
            → pushes tasks to executor queue → workers pull tasks
            (polling-based; scheduler is a potential bottleneck)

n8n:        Webhook arrives or cron fires → main process (or queue-mode)
            → executes workflow synchronously or enqueues to Redis
            (event-driven for webhooks, polling for cron)
```

### When Things Go Wrong

```
Temporal:   Worker crashes → History Service detects timeout
            → re-queues the activity → another worker picks it up
            → replays from last successful event. Zero data loss.

Airflow:    Task instance enters "zombie" state → scheduler reaps
            → task is retried (if configured). DAG run stays in DB.
            Weakness: scheduler loop failures can cause missed schedules.

n8n:        Worker crashes → execution may be lost (in single mode)
            → in queue mode, BullMQ can re-queue failed jobs
            → execution record marked as failed. No partial resumption.
```

---

## When to Pick Which

### Choose Temporal if:
- Your workflow encodes **core business logic** (payments, order fulfillment, user onboarding, identity)
- Workflows must **pause and resume** over hours, days, or weeks (waiting for human approval, external events)
- You need **guaranteed completion** — partial execution failure is unacceptable
- You're building **multi-step distributed transactions** (saga pattern)
- You're running **long-running AI agents** that need stateful, resumable execution
- Your team writes code in Go, Java, TypeScript, or Python and can absorb the learning curve

### Choose Apache Airflow if:
- Your workflows are **scheduled batch jobs** — daily ETL, hourly data syncs, nightly ML retraining
- You work with **data engineering toolchains** — dbt, Spark, Snowflake, BigQuery, Databricks
- You need **backfill** — re-running historical pipeline intervals after a fix
- Your team is Python-native and already familiar with the DAG mental model
- You need the largest ecosystem of **data tool integrations** (80+ official providers)
- Observability across scheduled batch jobs is a daily operational concern

### Choose n8n if:
- You need to **glue SaaS tools together** — Slack, GitHub, Salesforce, HubSpot, Gmail, Notion
- **Non-technical team members** need to build or modify workflows
- You want **rapid iteration** — new automations in minutes, not hours
- You need **webhook-driven automation** responding to external events
- Your data compliance requirements demand **self-hosted, on-prem** deployment (vs. Zapier)
- You're building **AI-assisted workflows** using the native LangChain/agent nodes
- Budget is a constraint and you want the lowest total cost at low-to-medium volume

---

## Operational Cost Snapshot

| Factor | Temporal (self-hosted) | Temporal Cloud | Airflow (self-hosted) | AWS MWAA | n8n (self-hosted) | n8n Cloud |
|--------|----------------------|----------------|----------------------|----------|-------------------|-----------|
| **Min infra cost** | $50–200/mo (VMs) | Usage-based ($$$) | $100–400/mo | $300+/mo | $5–20/mo (VPS) | From ~$20/mo |
| **Ops complexity** | High | Low | High | Low | Low | None |
| **Engineering to set up** | High (days) | Low (hours) | Medium (1–2 days) | Low (hours) | Low (<1 hour) | Zero |
| **Cost at high volume** | Scales with workers | Pay-per-use | Scales with workers | Expensive | Flat (server cost) | Scales up quickly |

---

## Honorable Mentions

### Prefect

- Python-native, modern Airflow alternative (founded 2018, 15,000+ stars)
- Simpler developer experience: `@flow` and `@task` decorators on any Python function
- Prefect Cloud separates orchestration plane (their cloud) from execution plane (your infra — "hybrid model")
- Excellent for small-to-medium Python teams wanting better DX than Airflow without Airflow's operational weight
- Smaller ecosystem than Airflow; fewer battle-tested production case studies at massive scale
- **Best fit:** Greenfield Python data projects, teams with 20–100 flows

### Dagster

- Asset-centric orchestration — workflows are defined around **data assets** (tables, ML models, reports) not just tasks
- Best-in-class **data lineage** and observability (knows what changed, what's stale)
- Deep integrations with dbt, Airbyte, Snowflake, Databricks
- Genuinely better developer experience than Airflow for new projects; better local testing story
- Smaller community than Airflow; fewer answers on StackOverflow
- **Best fit:** Modern data stack teams who care about data quality, lineage, and asset-centric thinking

### AWS Step Functions

- Fully managed serverless workflow orchestration on AWS
- Workflows defined in **Amazon States Language** (JSON/YAML state machine spec)
- Native integration with 200+ AWS services (Lambda, ECS, SNS, DynamoDB, Bedrock…)
- Two modes: **Standard Workflows** (durable, exactly-once, up to 1 year) and **Express Workflows** (high-volume, at-least-once, up to 5 minutes)
- Zero operational overhead — no servers to manage
- ⚠️ Hard vendor lock-in; cost scales with state transitions at high volume
- **Best fit:** AWS-native teams wanting zero-ops workflow orchestration with native AWS service glue

---

## Decision Flowchart

```
Is your workflow core business logic that MUST complete?
  ├── Yes → Does it span hours/days? Need saga/compensation?
  │           └── Yes → Temporal
  └── No → Is it a scheduled batch/ETL pipeline?
              ├── Yes → Data stack (dbt/Spark/Snowflake)?
              │           ├── Yes → Airflow (or Dagster/Prefect)
              │           └── Mostly AWS? → Step Functions
              └── No → Connecting SaaS tools / business APIs?
                          ├── Need code control / self-hosted → n8n
                          └── Mostly AWS serverless? → Step Functions
```

---

## Summary

| If you need… | Use |
|--------------|-----|
| Bulletproof distributed state machines | **Temporal** |
| Reliable data pipelines with complex dependency graphs | **Airflow** |
| Fast SaaS glue with minimal engineering overhead | **n8n** |
| Modern Airflow DX with asset lineage | **Dagster** |
| Simple Python orchestration, small team | **Prefect** |
| Zero-ops AWS-native orchestration | **Step Functions** |

The most common mistake is reaching for Temporal when n8n would suffice, or using Airflow for real-time API orchestration it wasn't designed for. Match the tool to the **failure mode** and **time horizon** of your workflows — not just the feature checklist.
