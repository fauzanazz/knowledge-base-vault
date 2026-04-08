---
title: "Data Engineering, Data Science, and ML for CTOs"
tags: [data-engineering, data-science, machine-learning, databases, nosql, dataops, team-building]
category: cto-leadership
sources:
  - "NoSQL Databases: a Survey and Decision Guidance"
  - "Evolutionary Database Design - Martin Fowler"
  - "Database Migrations Done Right"
  - "Building a data team at a mid-stage startup - Erik Bernhardsson"
  - "Building a data science team - Fast Data Science"
  - "How to Structure a Data Science Team - AltexSoft"
  - "Machine Learning Crash Course - Google"
  - "Deep Learning For Coders - fast.ai"
  - "Awesome Production Machine Learning"
last_updated: 2026-04-08
---

# Data Engineering, Data Science, and ML for CTOs

## Database Strategy

### NoSQL Decision Guide

The NoSQL landscape is confusing. This decision framework cuts through the hype:

| Database Type | Data Model | Best For | Examples |
|--------------|-----------|----------|----------|
| **Document** | JSON-like documents | Flexible schema, rapid iteration | MongoDB, CouchDB |
| **Key-Value** | Simple key → value | Caching, sessions, simple lookups | Redis, DynamoDB |
| **Wide-Column** | Column families | Time-series, IoT, high write throughput | Cassandra, HBase |
| **Graph** | Nodes + Edges | Relationships, social networks, recommendations | Neo4j, Amazon Neptune |
| **Search** | Inverted index | Full-text search, analytics | Elasticsearch, Meilisearch |

### Decision Matrix

| Requirement | Choose |
|------------|--------|
| ACID transactions required | PostgreSQL (relational) |
| Flexible schema, rapid prototyping | MongoDB or PostgreSQL JSONB |
| Massive write throughput | Cassandra, ScyllaDB |
| Complex relationships | Neo4j, PostgreSQL (recursive CTEs) |
| Full-text search | Elasticsearch, PostgreSQL (GIN indexes) |
| Key-value cache | Redis |
| Time-series data | TimescaleDB, InfluxDB |
| Global distribution | CockroachDB, Spanner, DynamoDB Global Tables |

### The Default Choice

**PostgreSQL** is the correct default for 90% of startups:
- Relational + JSON (JSONB) — best of both worlds
- ACID compliant
- Excellent full-text search (good enough to skip Elasticsearch for most cases)
- Extensions: PostGIS (geospatial), pgvector (AI embeddings), TimescaleDB (time-series)
- Massive community, mature tooling
- Scales vertically to impressive levels before you need to shard

**Switch away only when:** You have a specific, measured need that PostgreSQL can't meet.

### Evolutionary Database Design (Martin Fowler)

Key principles for database schema evolution:

1. **All database changes are migrations** — never modify schema manually
2. **Migrations are versioned and sequential** — like git commits for your schema
3. **Every migration has an UP and DOWN** — reversibility is critical
4. **Expand-Contract pattern** for breaking changes:
   ```
   Phase 1 (Expand): Add new column, keep old column
   Phase 2 (Migrate): Copy data, update app to use new column
   Phase 3 (Contract): Remove old column
   ```
5. **Never rename a column in one step** — it breaks running code during deployment

### Database Migration Best Practices

| Practice | Why |
|----------|-----|
| Run migrations in CI | Catch issues before production |
| Separate schema migration from data migration | Schema is fast, data is slow |
| Test migrations against production-sized data | That 2-second migration might take 2 hours with real data |
| Always backup before migrating production | Murphy's Law applies double to databases |
| Use online migration tools for large tables | `pt-online-schema-change`, `gh-ost` for MySQL; `pg_repack` for PostgreSQL |

## DataOps

DataOps applies DevOps principles to data pipelines:

### Core Practices

1. **Version control for data pipelines** — SQL, ETL scripts, dbt models in git
2. **Automated testing** — data quality checks, schema validation, row count assertions
3. **CI/CD for data** — automated deployment of pipeline changes
4. **Monitoring and alerting** — freshness, completeness, accuracy
5. **Self-service** — analysts can query and build dashboards without engineering

### Modern Data Stack

```
Sources → Ingestion → Warehouse → Transform → BI/Analytics
         (Fivetran,   (Snowflake,  (dbt)      (Metabase,
          Airbyte)     BigQuery,               Looker,
                       Redshift)               Superset)
```

**Reverse ETL:** Push transformed data back to operational systems (CRM, marketing tools) using Census, Hightouch.

## Building a Data Team

### Erik Bernhardsson's Framework (Mid-Stage Startup)

**When to hire your first data person:**
- You have > 6 months of product data
- Stakeholders are making decisions on gut feel
- Engineering is drowning in ad-hoc data requests

**Hiring sequence:**

| Order | Role | Why |
|-------|------|-----|
| 1st | **Analytics Engineer** | dbt, SQL, sets up the data warehouse and dashboards |
| 2nd | **Data Engineer** | Pipelines, infrastructure, data quality |
| 3rd | **Data Analyst** | Business intelligence, stakeholder support |
| 4th | **Data Scientist** | ML models, experimentation, prediction |
| 5th | **ML Engineer** | Productionize ML models |

**Key insight:** Most startups hire a data scientist first. This is wrong. Without data infrastructure (clean data, accessible warehouse), a data scientist can't do anything useful.

### Data Team Structures

| Model | Description | Best For |
|-------|------------|----------|
| **Centralized** | One data team serves all | Small company (< 50 eng) |
| **Embedded** | Data people sit in product teams | Mid-size (50-200 eng) |
| **Hub-and-spoke** | Central platform + embedded analysts | Large (200+ eng) |

### Managing Data Scientists vs Engineers

| Dimension | Software Engineers | Data Scientists |
|-----------|-------------------|-----------------|
| Output | Code, features | Insights, models |
| Definition of done | Tests pass, feature works | Statistical significance, business impact |
| Tools | IDE, git, CI/CD | Jupyter, notebooks, experiments |
| Iteration | PR → review → merge | Explore → hypothesis → validate |
| Risk | Technical debt | Invalid conclusions |

**Key management differences:**
- DS needs **research time** — don't fill their calendar with sprint tasks
- DS projects have **uncertain timelines** — exploration may lead nowhere
- DS needs **access to production data** — not a sanitized sandbox
- DS work must be **reproducible** — enforce version control for notebooks

## Machine Learning for CTOs

### When ML Makes Sense

ML is appropriate when:
1. The problem has a **clear metric** to optimize
2. You have **enough data** (thousands of examples minimum)
3. A **human can't easily write rules** for the decision
4. The **cost of errors** is manageable (or you can human-in-the-loop)

ML is NOT appropriate when:
- Simple rules/heuristics work (if/else > ML for most business logic)
- You have < 100 labeled examples
- The problem changes faster than you can retrain
- Explainability is critical and the model is a black box

### The ML Project Lifecycle

```
Problem Definition → Data Collection → Data Preparation → Model Training →
Evaluation → Deployment → Monitoring → Retraining
     ↑                                                          |
     └──────────────────────────────────────────────────────────┘
```

**Key CTO considerations:**

| Phase | CTO Question |
|-------|-------------|
| Problem Definition | "Is this actually an ML problem?" |
| Data Collection | "Do we have enough data? Is it clean? Legal?" |
| Model Training | "Buy vs build? API vs custom model?" |
| Deployment | "What's the latency requirement? Cost per inference?" |
| Monitoring | "How do we detect model degradation?" |
| Retraining | "How often? Automated or manual?" |

### Buy vs Build Decision

| Factor | Buy (API) | Build (Custom) |
|--------|-----------|----------------|
| Time to market | Days | Months |
| Cost (low volume) | Low | High |
| Cost (high volume) | High | Lower |
| Customization | Limited | Full |
| Data privacy | Vendor dependent | Full control |
| Maintenance | Vendor handles | Your team handles |

**Default recommendation:** Start with APIs (OpenAI, Anthropic, Google), build custom only when:
- API costs exceed training + serving costs
- You need fine-tuned performance on domain-specific data
- Data privacy/regulatory requirements demand it
- Latency requirements can't be met by API calls

### Production ML Pitfalls

1. **Training-serving skew** — model trained on different features than served
2. **Data drift** — real-world data distribution changes over time
3. **Feedback loops** — model predictions influence future training data
4. **Technical debt** — ML systems accumulate debt faster than traditional software
5. **Reproducibility** — "it worked on my laptop" but with ML models

### Ethical Considerations

- **Bias in training data** → biased predictions (hiring, lending, criminal justice)
- **Weapons of Math Destruction** (Cathy O'Neil) — opaque algorithms that reinforce inequality
- **Transparency** — can you explain why the model made a decision?
- **Consent** — are users aware their data trains models?

## Key Takeaways

1. **PostgreSQL is the default** — switch only with measured justification
2. **Evolutionary database design** — all changes are versioned migrations
3. **Hire analytics engineers before data scientists** — infrastructure before analysis
4. **DataOps = DevOps for data** — version control, testing, CI/CD for pipelines
5. **ML is a tool, not a strategy** — start with heuristics, upgrade to ML when data supports it
6. **Buy before build** for ML — APIs first, custom models only when justified
7. **Monitor ML models in production** — they degrade silently
