# System Design Knowledge Base

> Obsidian vault — 242 articles across 21 categories (deduplicated & restructured April 2026).

A comprehensive, interlinked knowledge base covering system design, distributed systems, databases, architecture patterns, infrastructure tools, MLOps/LLM serving, and AI fundamentals. Built for use with [Obsidian](https://obsidian.md) — all articles use `[[WikiLinks]]` for seamless navigation.

## 📚 Categories

| Category | Articles | Topics |
|----------|----------|--------|
| **system-design** | 36 | Core fundamentals — caching, load balancing, rate limiting, consistent hashing, scaling, CAP theorem, resiliency, fault isolation, idempotency, microservices, API gateway, service discovery, consensus |
| **messaging-and-events** | 20 | Message queues, consumer groups, topic partitioning, delivery semantics, event sourcing, CQRS, Saga, Outbox, fanout, Kappa/Lambda architecture, Redis pub/sub |
| **backend-architecture** | 17 | Clean/Hexagonal/DDD, modular monolith, serverless, service mesh, strangler fig, bulkhead, circuit breaker, BFF, API idempotency, REST APIs, event-driven, actor model |
| **payments-and-fintech** | 15 | Payment system, PSP integration, ledger, wallet, reconciliation, card schemes, stock exchange, matching engine, order manager, real-time bidding, payment security |
| **database** | 15 | ACID, replication, sharding, MVCC, NoSQL, NewSQL, optimistic/pessimistic locking, star schema, time-series DB, Dynamo-style, key-value store |
| **frontend-architecture** | 15 | Micro-frontends, Island Architecture, React Server Components, Resumability (Qwik), Streaming SSR, Hydration Strategies, Signals, Module Federation, Edge Rendering |
| **mlops** | 14 | vLLM Architecture, PagedAttention, KV Cache Management, Continuous Batching, Tensor/Pipeline Parallelism, Distributed LLM Serving, Prefix Caching, Speculative Decoding, Quantization |
| **fullstack-patterns** | 12 | Monorepo, Feature Flags, Blue-Green/Canary Deploys, GitOps, GraphQL, tRPC, WebSocket Architecture, Observability Stack, Trunk-Based Dev |
| **distributed-systems** | 12 | Consensus, consistency models, CRDTs, chain replication, leader election, logical clocks, failure detection, gossip protocol, CALM theorem |
| **monitoring-and-observability** | 11 | Metrics monitoring, alerting, dashboards, observability, SLOs, on-call engineering, distributed tracing, data aggregation/downsampling |
| **geospatial-and-maps** | 10 | Geocoding, geohash, quadtree, geospatial indexing, Google Maps, map tiling, proximity service, routing algorithms/tiles |
| **storage-systems** | 8 | Blob/block storage, CDN, file sync, delta sync, content deduplication, video transcoding |
| **data-structures** | 8 | LSM Trees, Merkle Tree, Skip List, Trie, Vector Clock, WAL, Redis Sorted Sets, Order Book |
| **infrastructure-tools** | 7 | Message Brokers (RabbitMQ/Kafka/SQS/NATS/Pulsar), Redis, Web Servers (Apache/Nginx/HAProxy/Caddy/Traefik), HashiCorp (Terraform/Vault/Consul/Nomad), Cloud Platforms (AWS/GCP/Azure), Collaboration & DevOps, Enterprise Linux & Containers |
| **networking** | 7 | DNS, HTTP, TCP, WebSocket, Email Protocols, FIX Protocol, TLS Security |
| **real-time-systems** | 7 | Chat system, online presence, nearby friends, gaming leaderboard, notification, push notification, news feed |
| **ai-fundamentals** | 6 | Transformer Architecture, Attention Mechanisms, Modern Reasoning Methods, RLHF Alignment, Autonomous AI Research, Scaling Laws |
| **algorithms** | 6 | Base62 encoding, rate limiting (token/leaking bucket, sliding window), Twitter Snowflake, vector clocks |
| **case-studies** | 6 | YouTube, Google Drive, Hotel Reservation, URL Shortener, Distributed Email, Ad Click Aggregation |
| **software-engineering** | 5 | Testing, CI/CD, Formal Verification, Maintainability, Test Doubles |
| **search-and-crawling** | 5 | Search Autocomplete, Web Crawler, URL Frontier, Email Search, robots.txt |

## 🔗 Sources

- **System Design Interview** — Alex Xu (Vol. 1 & 2)
- **Understanding Distributed Systems** — Roberto Vitillo
- **vLLM/MLOps Research** (2026) — SOSP '23 PagedAttention paper, vLLM docs, distributed inference research
- **Infrastructure Tools Research** (2026) — vendor docs, benchmarks, community comparisons

## 🚀 Usage

### As Obsidian Vault
```bash
git clone git@github.com:fauzanazz/knowledge-base-vault.git
# Open the cloned folder in Obsidian as a vault
```

### CLI Query (with `kb` tool)
```bash
cd /path/to/system-design
kb search "rate limiter"                        # Full-text search
kb query "how does consistent hashing work?"    # Semantic Q&A
```

## 📁 Structure

```
.
├── _index.md                       # Master index of all articles
├── ai-fundamentals/                # Transformers, attention, RLHF, scaling laws
├── algorithms/                     # Encoding, rate limiting, ID generation
├── backend-architecture/           # Clean/Hex/DDD, patterns, REST, API design
├── case-studies/                   # Full system designs (YouTube, Google Drive, etc.)
├── data-structures/                # Trees, lists, logs, sorted sets
├── database/                       # SQL, NoSQL, replication, sharding, locking
├── distributed-systems/            # Consensus, consistency, fault tolerance
├── frontend-architecture/          # SSR, hydration, micro-frontends, signals
├── fullstack-patterns/             # DevOps, deployment, observability, GitOps
├── geospatial-and-maps/            # Geocoding, geohash, routing, map tiling
├── infrastructure-tools/           # RabbitMQ, Kafka, Redis, HashiCorp, AWS/GCP/Azure
├── messaging-and-events/           # Queues, event sourcing, CQRS, Saga, Outbox
├── mlops/                          # LLM serving, vLLM, inference optimization
├── monitoring-and-observability/   # Metrics, alerting, dashboards, SLOs, tracing
├── networking/                     # Protocols (DNS, HTTP, TCP, WS, TLS)
├── payments-and-fintech/           # Payment systems, ledger, wallet, PSP, exchange
├── real-time-systems/              # Chat, presence, leaderboard, notifications
├── search-and-crawling/            # Autocomplete, web crawler, URL frontier
├── software-engineering/           # Testing, CI/CD, verification
├── storage-systems/                # Blob/block storage, CDN, file sync
└── system-design/                  # Core fundamentals (caching, scaling, consensus)
```

## 📝 License

Personal knowledge base. Not for redistribution.
