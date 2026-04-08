# Infrastructure Tools — Category Overview

> **Last Updated:** April 2026  
> **Audience:** Engineers, architects, and technical leads making infrastructure decisions  
> **Scope:** Category-level index, landscape maps, stack recommendations, and architecture pattern guidance

---

## Table of Contents

1. [Articles in This Category](#articles-in-this-category)
2. [Technology Landscape Map](#technology-landscape-map)
3. [How the Layers Connect](#how-the-layers-connect)
4. [Stack Recommendations by Company Size](#stack-recommendations-by-company-size)
5. [Common Architecture Patterns](#common-architecture-patterns)
   - [Monolith](#monolith)
   - [Microservices](#microservices)
   - [Serverless](#serverless)
6. [Tool Cross-Reference Matrix](#tool-cross-reference-matrix)
7. [Further Reading](#further-reading)

---

## Articles in This Category

### [Cloud Platforms — AWS vs GCP vs Azure](./cloud-platforms.md)
A comprehensive comparison of the three major hyperscalers covering compute (VMs, serverless functions), object storage, relational and NoSQL databases, networking, managed Kubernetes (EKS, GKE, AKS), identity & security, observability, pricing models, and multi-cloud strategy. Includes explicit guidance on when to choose each provider and a detailed quick-reference matrix.

**Key topics:** EC2 / Compute Engine / Azure VMs, Lambda / Cloud Run / Azure Functions, S3 / GCS / Azure Blob, Aurora / AlloyDB / Azure SQL, DynamoDB / Bigtable / CosmosDB, EKS / GKE / AKS, IAM & RBAC, cost optimisation.

---

### [HashiCorp Suite — Terraform, Vault, Consul, Nomad & More](./hashicorp-suite.md)
A deep-dive into the HashiCorp product portfolio, covering each tool's philosophy, architecture, and operational guidance. Includes the impact of HashiCorp's 2023 BUSL licence change and the OpenTofu fork.

**Key topics:**
- **Terraform / OpenTofu** — declarative IaC, HCL deep dive, state management, comparison with Pulumi / CloudFormation / CDK
- **Vault** — secrets management, dynamic credentials, encryption-as-a-service (Transit engine), Vault vs AWS Secrets Manager
- **Consul** — service discovery, health checking, service mesh (Consul Connect), comparison with etcd and ZooKeeper
- **Nomad** — workload orchestration, comparison with Kubernetes, mixed workload scheduling (containers + VMs + binaries)
- **Packer** — immutable machine image automation across cloud providers
- **Vagrant** — reproducible developer environments (and when to prefer Dev Containers instead)
- **Waypoint** — unified build/deploy/release abstraction across platforms

---

### [Redis — In-Depth Architecture & Use Cases](./redis.md)
A thorough examination of Redis as an in-memory data platform covering its single-threaded event loop, all core data structures, persistence mechanisms (RDB + AOF), high availability patterns, and horizontal scaling.

**Key topics:** Single-threaded architecture, Strings / Lists / Hashes / Sets / Sorted Sets / Streams / HyperLogLog / Geo, RDB snapshots vs AOF replication, Redis Sentinel (HA), Redis Cluster (sharding), caching patterns, session stores, pub/sub, rate limiting (fixed + sliding window), leaderboards, distributed locks (Redlock), Redis vs Memcached, Redis vs DynamoDB/DAX, Redis Streams vs Kafka, and when *not* to use Redis.

---

### [Message Brokers — RabbitMQ, Kafka, SQS, NATS, Pulsar](./message-brokers.md)
A comprehensive guide to asynchronous messaging infrastructure, covering the spectrum from traditional queues to event streaming platforms. Includes head-to-head comparisons, performance benchmarks, ordering guarantees, and a five-question decision framework.

**Key topics:**
- **RabbitMQ** — AMQP protocol, exchange types (direct/fanout/topic/headers), publisher confirms, quorum queues, use cases for task queues and RPC
- **Apache Kafka** — distributed commit log architecture, topics/partitions/offsets, consumer groups, KRaft (post-ZooKeeper), Kafka Streams, ksqlDB, Kafka Connect
- **RabbitMQ vs Kafka** — paradigm differences, throughput, latency, message replay, ordering guarantees
- **Amazon SQS / SNS** — fully managed queuing and fan-out on AWS, standard vs FIFO queues, SNS+SQS fan-out pattern
- **NATS / JetStream** — ultra-lightweight messaging for edge and IoT, JetStream persistence
- **Apache Pulsar** — stateless brokers + BookKeeper storage, compute/storage separation, multi-tenancy, geo-replication
- **Apache ActiveMQ** — JMS compliance, Java/JEE ecosystem integration

---

### [Web Servers & Proxies — Nginx, Apache, HAProxy, Caddy, Traefik](./web-servers-and-proxies.md)
A detailed comparison of the major web server and proxy technologies, covering architecture, concurrency models, configuration paradigms, and concrete use-case guidance.

**Key topics:**
- **Apache HTTP Server** — Prefork/Worker/Event MPMs, `.htaccess` per-directory config, mod_php, strengths for shared hosting and legacy PHP
- **Nginx** — async event-driven architecture, epoll/kqueue, low memory footprint at high concurrency, reverse proxy, static file serving via `sendfile()`
- **Apache vs Nginx** — head-to-head on memory, performance, PHP handling, config model
- **Reverse proxy patterns** — SSL termination, load balancing algorithms (round-robin, least-connections, IP hash, weighted), connection pooling / upstream keepalive, health checks
- **HAProxy** — Layer 4/7 load balancing, ACL-based routing, runtime API, statistics dashboard; best-in-class for pure load balancing
- **Caddy** — automatic HTTPS via Let's Encrypt, Caddyfile syntax, JSON API for dynamic config
- **Traefik** — cloud-native dynamic proxy, Docker/Kubernetes auto-discovery, middleware composability, Ingress controller

---

## Technology Landscape Map

The following map shows where each tool in this category sits within a complete infrastructure stack, reading from developer workflow at the top to production runtime at the bottom.

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                     INFRASTRUCTURE TOOLS LANDSCAPE                          ║
╠══════════════════════════════════════════════════════════════════════════════╣
║                                                                              ║
║  ┌─────────────────────────────────────────────────────────────────────┐    ║
║  │  DEVELOPER ENVIRONMENT                                               │    ║
║  │  Vagrant (full VM) · Dev Containers · GitHub Codespaces             │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │ code pushed                                ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  IMAGE & ARTIFACT BUILDING                                           │    ║
║  │  Packer (AMI / VM images) · Docker / OCI containers                 │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  INFRASTRUCTURE PROVISIONING  (IaC)                                  │    ║
║  │  Terraform / OpenTofu · Pulumi · CloudFormation / CDK               │    ║
║  │  ──────────────────────────────────────────────────────────────────  │    ║
║  │  Provisions ↓                                                        │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  CLOUD FOUNDATION                                                    │    ║
║  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │    ║
║  │  │  AWS          │  │  GCP          │  │  Azure                   │  │    ║
║  │  │  EC2/EKS/RDS  │  │  GCE/GKE/SQL  │  │  VMs/AKS/SQL DB         │  │    ║
║  │  │  Lambda/S3    │  │  Cloud Run/   │  │  Functions/Blob         │  │    ║
║  │  │  DynamoDB     │  │  BigQuery/GCS  │  │  CosmosDB               │  │    ║
║  │  └──────────────┘  └──────────────┘  └──────────────────────────┘  │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  WORKLOAD ORCHESTRATION                                              │    ║
║  │  Kubernetes (EKS/GKE/AKS) · Nomad · AWS ECS/Fargate                │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  TRAFFIC INGRESS & LOAD BALANCING                                    │    ║
║  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌─────────────┐  │    ║
║  │  │  Nginx     │  │  HAProxy   │  │  Traefik   │  │  Caddy      │  │    ║
║  │  │  (proxy +  │  │  (TCP/HTTP │  │  (K8s      │  │  (auto TLS) │  │    ║
║  │  │  web svr)  │  │  LB)       │  │  ingress)  │  │             │  │    ║
║  │  └────────────┘  └────────────┘  └────────────┘  └─────────────┘  │    ║
║  │  Apache (shared hosting / legacy PHP)                               │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  APPLICATION SERVICES                                                │    ║
║  │                                                                      │    ║
║  │  ┌──────────────────────────────────────────────────────────────┐   │    ║
║  │  │  CACHING & IN-MEMORY DATA                                    │   │    ║
║  │  │  Redis (cache · sessions · leaderboards · pub/sub · streams) │   │    ║
║  │  │  Memcached (simple string cache)                             │   │    ║
║  │  └──────────────────────────────────────────────────────────────┘   │    ║
║  │                                                                      │    ║
║  │  ┌──────────────────────────────────────────────────────────────┐   │    ║
║  │  │  ASYNC MESSAGING & EVENT STREAMING                           │   │    ║
║  │  │  Apache Kafka (high-throughput event log, CDC, pipelines)    │   │    ║
║  │  │  RabbitMQ (task queues, complex routing, RPC)                │   │    ║
║  │  │  Amazon SQS/SNS (managed, AWS-native fan-out)                │   │    ║
║  │  │  NATS JetStream (edge, IoT, lightweight streaming)           │   │    ║
║  │  │  Apache Pulsar (multi-tenant, geo-replicated streaming)      │   │    ║
║  │  └──────────────────────────────────────────────────────────────┘   │    ║
║  └──────────────────────────────┬──────────────────────────────────────┘    ║
║                                 │                                            ║
║  ┌──────────────────────────────▼──────────────────────────────────────┐    ║
║  │  SERVICE NETWORKING & SECURITY                                       │    ║
║  │  ┌──────────────────────┐   ┌───────────────────────────────────┐   │    ║
║  │  │  Consul              │   │  HashiCorp Vault                   │   │    ║
║  │  │  - Service discovery │   │  - Secrets management             │   │    ║
║  │  │  - Health checking   │   │  - Dynamic credentials            │   │    ║
║  │  │  - Service mesh mTLS │   │  - Encryption as a service        │   │    ║
║  │  │  - KV store          │   │  - PKI / TLS cert issuance        │   │    ║
║  │  └──────────────────────┘   └───────────────────────────────────┘   │    ║
║  └──────────────────────────────────────────────────────────────────────┘    ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## How the Layers Connect

Understanding the *interactions* between tools is as important as understanding each tool individually.

### Terraform → Cloud Platforms
Terraform provisions VPCs, EC2/GCE/Azure VMs, managed Kubernetes clusters (EKS/GKE/AKS), RDS databases, S3 buckets, and IAM roles on all three major cloud providers. It is the primary IaC tool for declaring **what** cloud infrastructure exists. See [cloud-platforms.md](./cloud-platforms.md) and [hashicorp-suite.md](./hashicorp-suite.md).

### Packer → Terraform → Cloud Platforms
Packer builds hardened, pre-configured machine images (AMIs, GCP Machine Images, Azure Managed Images) incorporating application code and OS configuration. Terraform then references the latest image ID (often stored in SSM Parameter Store or equivalent) to deploy immutable EC2/GCE/Azure VM fleets. This is the **immutable infrastructure** pattern. See [hashicorp-suite.md](./hashicorp-suite.md).

### Vault → All Application Layers
Vault acts as the credential authority for the entire stack. Terraform uses the Vault provider to provision secrets. Kubernetes pods authenticate via service account JWTs to receive short-lived database credentials. Nomad jobs use embedded templates to fetch Vault secrets at launch time. Applications request TLS certificates from Vault's PKI engine. Nothing stores long-lived credentials. See [hashicorp-suite.md](./hashicorp-suite.md).

### Consul → Nomad / Kubernetes
Consul maintains the service catalogue for all running workloads. Nomad natively integrates with Consul for service registration and health checking. In Kubernetes environments, Consul Connect can provide the service mesh layer alongside or instead of Istio. Traefik and Nginx can both discover upstreams via Consul DNS. See [hashicorp-suite.md](./hashicorp-suite.md).

### Web Servers / Proxies → Application Services → Message Brokers
Incoming traffic flows through Nginx or Traefik (ingress), reaches application services, which write events to Kafka or RabbitMQ for downstream async processing. Redis sits in the critical path for caching (cache-aside pattern), session storage, and rate limiting at the proxy or application layer. See [web-servers-and-proxies.md](./web-servers-and-proxies.md), [redis.md](./redis.md), and [message-brokers.md](./message-brokers.md).

### Cloud Platform Managed Services vs Self-Hosted Tools
Many tools have cloud-native equivalents that replace self-managed infrastructure:

| Self-Managed Tool | AWS Equivalent | GCP Equivalent | Azure Equivalent |
|-------------------|---------------|----------------|-----------------|
| Redis (self-hosted) | ElastiCache for Redis | Memorystore for Redis | Azure Cache for Redis |
| Apache Kafka | Amazon MSK | Google Managed Kafka | Azure Event Hubs (Kafka-compatible) |
| RabbitMQ | Amazon MQ for RabbitMQ | — | Azure Service Bus |
| HashiCorp Vault | AWS Secrets Manager | Secret Manager | Azure Key Vault |
| Nginx Ingress | AWS ALB / CloudFront | Cloud Load Balancing | Azure Front Door |
| Consul | AWS Cloud Map | — | Azure Service Fabric |
| Terraform state backend | S3 + DynamoDB | GCS | Azure Blob + Leasing |

See [cloud-platforms.md](./cloud-platforms.md) for full service comparisons.

---

## Stack Recommendations by Company Size

These recommendations are opinionated starting points. Every organisation's context differs; treat these as informed defaults, not mandates.

### 🚀 Startup (< 10 developers)

**Priorities:** Minimal operational overhead, fast iteration, low cost, avoid premature complexity.

```
Cloud:              AWS (broadest ecosystem, best startup credits via AWS Activate)
                    → or GCP if ML/data is core to the product
                    → or Azure if B2B enterprise sales requires it

IaC:                Terraform (or OpenTofu) with S3 + DynamoDB remote state
                    → Keep it simple: one root module per environment
                    → Skip Terragrunt until you have >3 environments

Compute:            AWS ECS/Fargate (no Kubernetes operational overhead)
                    → Step up to EKS/GKE only when Fargate limits bite

Web / Proxy:        AWS ALB (managed, zero ops) or Caddy (single binary, auto-TLS)
                    → Skip Apache; skip HAProxy unless TCP LB is needed

Caching:            ElastiCache for Redis (AWS managed) or Redis Cloud Free tier
                    → Don't manage your own Redis cluster

Messaging:          Amazon SQS + SNS for simple async tasks
                    → Add RabbitMQ (CloudAMQP hosted) if complex routing needed
                    → Skip Kafka — operational overhead is not justified at this scale

Secrets:            AWS Secrets Manager (zero-ops, integrates with ECS/Lambda natively)
                    → Vault only if multi-cloud or need dynamic DB credentials at day 1

Service Discovery:  AWS Cloud Map or DNS-based (no Consul yet)

Image Building:     Docker only; skip Packer until you're managing VM fleets

Developer Envs:     Dev Containers (VS Code) or GitHub Codespaces
                    → Skip Vagrant
```

**What to avoid:** Kubernetes, self-managed Kafka, self-managed Vault, multi-cloud.

**Golden rule:** Every managed service you use instead of self-managing is an engineer freed to build product.

---

### 🏢 Mid-Size (10–100 developers)

**Priorities:** Scalability, security maturity, team autonomy, platform engineering foundations.

```
Cloud:              AWS primary (or GCP if K8s/data-heavy)
                    → Consider GCP BigQuery + Looker as analytics layer
                      even on AWS-primary stack

IaC:                Terraform / OpenTofu with Terragrunt for DRY multi-environment
                    → Separate state per environment (dev/staging/prod)
                    → Terraform in CI: plan on PR, apply on merge
                    → HCP Terraform or Atlantis for team collaboration

Compute:            Kubernetes (EKS or GKE) for application workloads
                    → EKS with Karpenter for cost-efficient node provisioning
                    → GKE Autopilot for teams that want managed nodes
                    → Keep Fargate for lightweight batch/Lambda for functions

Web / Proxy:        Nginx Ingress Controller (on Kubernetes)
                    → or Traefik for auto-discovery in dynamic K8s environments
                    → HAProxy for any TCP load balancing (databases, Redis)

Caching:            ElastiCache Redis with Sentinel or Cluster mode
                    → Evaluate Redis Cluster when single node > 50GB
                    → Use Redis for sessions, rate limiting, and caching

Messaging:          RabbitMQ (CloudAMQP or AWS AmazonMQ) for task queues
                    → Add Kafka (AWS MSK) when you need:
                      - Event replay / CDC pipelines
                      - > 3 independent consumer teams on same stream
                      - Audit log requirements

Secrets:            HashiCorp Vault (HCP Vault managed, or self-hosted on K8s with HA)
                    → Enable Kubernetes auth method + dynamic DB credentials
                    → Vault is justified once you have >5 services sharing secrets

Service Discovery:  Kubernetes DNS for K8s-native services
                    → Consul for hybrid (K8s + VMs), multi-datacenter, or Nomad workloads

Image Building:     Packer for golden AMIs (hardened OS, agent pre-installed)
                    → Feed AMI IDs to Terraform via SSM Parameter Store

Developer Envs:     Dev Containers + GitHub Codespaces for standardised environments
```

**Emerging concerns at this stage:**
- **Cost visibility:** Implement cloud cost allocation tags via Terraform modules. Add AWS Cost Explorer or Infracost in CI.
- **Security posture:** Enable Vault dynamic secrets for all database access. Rotate all static credentials.
- **Blast radius:** Separate AWS accounts per environment (via AWS Organizations). Use Terraform workspaces or directory-per-env.

---

### 🏦 Enterprise (100+ developers)

**Priorities:** Multi-team autonomy, compliance, security by default, multi-region resiliency, platform standardisation.

```
Cloud:              Multi-cloud by design or AWS/Azure primary with
                    GCP BigQuery as analytical layer
                    → Avoid true application-level multi-cloud (too complex)
                    → Use Terraform to abstract cloud differences

IaC:                Terraform / OpenTofu with Terragrunt or internal platform modules
                    → Internal module registry (Terraform Cloud, Spacelift, or Atlantis)
                    → Strict policy-as-code (Sentinel policies or OPA / Conftest)
                    → Automated cost estimation in PR pipelines (Infracost)
                    → Separate state hierarchy: foundation → platform → application

Compute:            Kubernetes (EKS/GKE/AKS) for all containerised workloads
                    → Nomad for mixed container/VM/binary workloads or batch
                    → Multiple clusters: per-region, per-BU, or per-criticality tier
                    → Cluster fleet management: EKS Anywhere / GKE Enterprise (Anthos) / AKS + Azure Arc

Web / Proxy:        Nginx or HAProxy as ingress per cluster
                    → Global CDN (CloudFront / GCP Cloud CDN / Azure Front Door)
                    → WAF at CDN edge (AWS WAF, Cloudflare Enterprise)
                    → Traefik for teams with high service churn in K8s namespaces

Caching:            Redis Cluster (ElastiCache / Memorystore / Azure Cache)
                    → Multiple Redis tiers (hot cache, session, rate limit)
                    → Redis Enterprise for full encryption-at-rest, Active-Active geo-replication

Messaging:          Apache Kafka (self-managed on K8s with Strimzi, or Confluent Cloud / MSK)
                    → Schema Registry (Confluent or Apicurio) for contract enforcement
                    → Kafka Connect for all CDC and data pipeline integration
                    → RabbitMQ retained for teams needing low-latency task queues
                    → Apache Pulsar if multi-tenancy and independent compute/storage scaling required

Secrets:            HashiCorp Vault Enterprise (HA cluster, DR replication, namespaces)
                    → Dynamic secrets for ALL database access (no static DB passwords)
                    → Vault namespaces for team/BU isolation
                    → Vault as PKI for internal mTLS certificates
                    → Break-glass procedures and Vault audit logging to SIEM

Service Discovery:  Consul Enterprise for multi-datacenter service registry
                    → Consul Connect for service mesh mTLS across VM + K8s workloads
                    → Evaluate Istio for K8s-only, feature-rich L7 mesh

Image Building:     Packer golden images as mandatory baseline for all VM workloads
                    → Image pipeline: Packer → security scan → golden AMI → Terraform
                    → Images rebuild on CVE patches via automated pipeline

Developer Envs:     Internal Developer Platform (IDP) built on top of K8s
                    → Self-service environment provisioning (Backstage, Humanitec)
                    → Dev Containers for local development standardisation
```

**Critical enterprise concerns:**
- **Compliance:** Vault audit logs → SIEM. CloudTrail / Audit Logs → security platform. All secrets rotated automatically.
- **Resiliency:** Multi-region active-active or active-passive for critical services. Kafka geo-replication. Redis Active-Active (Enterprise).
- **Cost governance:** FinOps practice, chargeback by team/product using cloud cost allocation tags enforced by Terraform Sentinel policies.
- **Platform engineering:** Dedicated Platform Engineering team owns the IaC modules, golden images, Vault policies, Kafka cluster, and developer experience abstractions.

---

## Common Architecture Patterns

### Monolith

A monolith runs as a single deployable unit. The infrastructure stack is simpler but must scale vertically or as identical clones behind a load balancer.

```
                          ┌─────────────────────────────┐
  Internet                │    Load Balancer             │
  ──────────────────────▶ │    (Nginx / HAProxy / ALB)   │
                          └───────────┬─────────────────┘
                                      │
             ┌────────────────────────▼──────────────────┐
             │                                            │
    ┌────────▼─────────┐         ┌────────▼─────────┐    │
    │  App Instance 1  │         │  App Instance 2  │    │
    │  (Monolith)      │         │  (Monolith)      │    │
    └────────┬─────────┘         └────────┬─────────┘    │
             └─────────────┬──────────────┘              │
                           │                             │
             ┌─────────────▼──────────────────────────┐  │
             │  Redis (sessions + cache)               │  │
             └─────────────────────────────────────────┘  │
             ┌─────────────────────────────────────────┐  │
             │  Primary Database (RDS / PostgreSQL)     │  │
             └─────────────────────────────────────────┘  │
             └────────────────────────────────────────────┘
```

**Typical tool choices for a monolith:**

| Layer | Typical Tools |
|-------|--------------|
| **Web / Proxy** | Nginx (reverse proxy + static files) or Caddy (auto-TLS, simpler config) |
| **Load balancer** | AWS ALB, Nginx, or HAProxy for TCP-level balancing |
| **Caching** | Redis for session storage and read-through caching; [redis.md](./redis.md) |
| **Messaging** | Amazon SQS or RabbitMQ for background jobs (email, processing); [message-brokers.md](./message-brokers.md) |
| **Cloud** | AWS (most common), single region; [cloud-platforms.md](./cloud-platforms.md) |
| **IaC** | Terraform with a flat module structure; single state file per environment |
| **Secrets** | AWS Secrets Manager or HashiCorp Vault (basic KV); [hashicorp-suite.md](./hashicorp-suite.md) |
| **Images** | Docker for containers; Packer for VM-deployed monoliths |

**What you typically do NOT need in a monolith:**
- Kubernetes (ECS/Fargate or plain EC2 Auto Scaling is sufficient)
- Apache Kafka (SQS or RabbitMQ handles async tasks at this scale)
- Consul service discovery (DNS-based discovery or hardcoded internal hostnames work fine)
- Nomad or Vault dynamic secrets (Secrets Manager rotation is sufficient)

---

### Microservices

Microservices decompose functionality into independent, separately deployable services. Infrastructure complexity rises substantially — each service has its own deployment pipeline, scaling profile, and often its own data store.

```
                                ┌──────────────────────────┐
  Internet                      │   CDN / Global LB         │
  ───────────────────────────▶  │   (CloudFront / GCP CDN)  │
                                └──────────┬───────────────┘
                                           │
                                ┌──────────▼───────────────┐
                                │   API Gateway / Ingress   │
                                │   (Nginx / Traefik /      │
                                │    AWS API Gateway)       │
                                └──┬───────┬────────┬──────┘
                                   │       │        │
                         ┌─────────▼─┐ ┌───▼─────┐ ┌▼────────────┐
                         │ Service A │ │Service B │ │  Service C  │
                         │ (orders)  │ │(payments)│ │(inventory)  │
                         └─────┬─────┘ └────┬────┘ └──────┬──────┘
                               │             │              │
                     ┌─────────▼─────────────▼──────────────▼──────┐
                     │         Message Bus (Apache Kafka)            │
                     │         - orders topic                        │
                     │         - payment-events topic                │
                     │         - inventory-events topic              │
                     └─────────┬─────────────┬──────────────┬──────┘
                               │             │              │
                     ┌─────────▼──┐ ┌────────▼──┐ ┌────────▼──────┐
                     │ Notif. Svc │ │Analytics  │ │ Fulfillment   │
                     │ (consumer) │ │ (consumer)│ │ (consumer)    │
                     └────────────┘ └───────────┘ └───────────────┘

  Cross-cutting concerns (every service):
  ┌──────────────────┐  ┌─────────────────────┐  ┌──────────────────────┐
  │  Redis           │  │  Consul / K8s DNS   │  │  Vault               │
  │  (per-service    │  │  (service discovery │  │  (dynamic DB creds,  │
  │   caches,        │  │   + health checks)  │  │   PKI, secrets)      │
  │   rate limiting) │  └─────────────────────┘  └──────────────────────┘
  └──────────────────┘
```

**Typical tool choices for microservices:**

| Layer | Typical Tools |
|-------|--------------|
| **Orchestration** | Kubernetes (EKS/GKE/AKS) or Nomad for mixed workloads |
| **Ingress** | Traefik (K8s auto-discovery) or Nginx Ingress Controller; [web-servers-and-proxies.md](./web-servers-and-proxies.md) |
| **Service mesh** | Consul Connect (multi-environment), Istio or Linkerd (K8s-only) |
| **Service discovery** | Consul or Kubernetes DNS; [hashicorp-suite.md](./hashicorp-suite.md) |
| **Event streaming** | Apache Kafka for domain events, CDC, and audit logs; [message-brokers.md](./message-brokers.md) |
| **Task queues** | RabbitMQ alongside Kafka for per-service task queues |
| **Caching** | Redis (per-service caches, shared rate-limit counters); [redis.md](./redis.md) |
| **Secrets** | HashiCorp Vault with Kubernetes auth, dynamic DB credentials |
| **IaC** | Terraform + Terragrunt for multi-service, multi-environment infrastructure |
| **Cloud** | AWS, GCP, or Azure; [cloud-platforms.md](./cloud-platforms.md) |

**Key microservices-specific concerns:**
- **Inter-service communication:** Synchronous (gRPC, REST) + asynchronous (Kafka topics). Design for at-least-once delivery and idempotent consumers.
- **Data isolation:** Each service owns its own database. Kafka CDC (via Debezium) for cross-service data sharing without tight coupling.
- **Distributed tracing:** OpenTelemetry + Jaeger/Tempo alongside Kafka and Redis for end-to-end request visibility.
- **Config & secrets sprawl:** Without Vault, secrets become unmanageable across dozens of services. Dynamic credentials prevent the rotating-credentials problem.

---

### Serverless

Serverless architectures replace always-on servers with functions that execute on demand, scaling to zero when idle. Infrastructure management is minimal but the tool selection is constrained to managed services.

```
                         ┌──────────────────────────────────────┐
  Client Request         │  API Gateway (AWS API GW / Cloud     │
  ─────────────────────▶ │  Endpoints / Azure APIM)             │
                         └──────────────────┬───────────────────┘
                                            │
               ┌────────────────────────────▼─────────────────────────┐
               │               Function Layer                          │
               │  ┌──────────────┐  ┌──────────────┐  ┌────────────┐ │
               │  │  Lambda /    │  │  Lambda /    │  │ Lambda /   │ │
               │  │  Cloud Run / │  │  Cloud Run / │  │ Cloud Run  │ │
               │  │  Azure Func  │  │  Azure Func  │  │            │ │
               │  └──────┬───────┘  └──────┬───────┘  └─────┬──────┘ │
               └─────────┼─────────────────┼────────────────┼────────┘
                         │                 │                │
          ┌──────────────▼──┐  ┌───────────▼───┐  ┌────────▼────────┐
          │  DynamoDB /     │  │  Amazon SQS/  │  │  ElastiCache    │
          │  Firestore /    │  │  SNS /        │  │  Redis /        │
          │  CosmosDB       │  │  Event Bridge │  │  Upstash Redis  │
          └─────────────────┘  └───────────────┘  └─────────────────┘
                                                   (serverless Redis)

  Infrastructure managed via:
  ┌──────────────────────────────────────────────────────────────┐
  │  Terraform (IaC for all managed service configuration)       │
  │  AWS SAM / Serverless Framework / CDK (function deployment)  │
  │  AWS Secrets Manager / GCP Secret Manager / Azure Key Vault  │
  └──────────────────────────────────────────────────────────────┘
```

**Typical tool choices for serverless:**

| Layer | Typical Tools |
|-------|--------------|
| **Compute** | AWS Lambda, GCP Cloud Run, Azure Functions; [cloud-platforms.md](./cloud-platforms.md) |
| **Web / Proxy** | AWS API Gateway, CloudFront; no Nginx/HAProxy needed |
| **Caching** | AWS DAX (for DynamoDB), ElastiCache Redis, or Upstash (serverless Redis over HTTP) |
| **Messaging** | Amazon SQS + SNS (fully managed, integrates natively with Lambda); [message-brokers.md](./message-brokers.md) |
| **Event streaming** | Amazon Kinesis or EventBridge for lightweight streaming; Kafka only if pre-existing |
| **Secrets** | AWS Secrets Manager or SSM Parameter Store (no Vault — operational overhead unjustified) |
| **IaC** | Terraform for managed service infrastructure + AWS SAM / Serverless Framework / CDK for functions |
| **Service discovery** | No Consul — DNS-based or AWS Service Connect |
| **Images** | No Packer — container images for Lambda (or ZIP deployments); no VM image building |

**What you typically do NOT need in serverless:**
- Nginx, HAProxy, Traefik — API Gateway replaces all ingress concerns
- HashiCorp Vault — cloud-native secrets managers handle rotation with Lambda integration
- Consul — service connectivity is handled by cloud-native mechanisms (VPC endpoints, service endpoints)
- Kubernetes or Nomad — the platform manages all scheduling and scaling
- Redis Sentinel / Redis Cluster — use cloud-managed Redis (ElastiCache) or serverless Redis (Upstash)
- Apache Kafka — unless the architecture has a pre-existing Kafka investment; Kinesis or SQS are simpler

**Serverless constraints to plan for:**
- **Cold starts:** Lambda/Cloud Run functions have startup latency. Use provisioned concurrency (Lambda) or minimum instances (Cloud Run) for latency-sensitive paths.
- **Execution limits:** Lambda max 15 minutes, Azure Functions Consumption Plan max 10 minutes. Long-running work needs Step Functions orchestration or async patterns.
- **Stateless by design:** All state must be externalised (DynamoDB, Redis, S3). Redis for ephemeral session data; DynamoDB for persistent state.
- **Vendor lock-in is real:** Serverless architectures are the most cloud-specific of the three patterns. Migration between providers requires significant rework.

---

## Tool Cross-Reference Matrix

Use this matrix to quickly identify which tools are relevant for a given concern.

| Concern | Relevant Articles |
|---------|------------------|
| Provisioning cloud infrastructure (IaC) | [hashicorp-suite.md](./hashicorp-suite.md), [cloud-platforms.md](./cloud-platforms.md) |
| Choosing a cloud provider | [cloud-platforms.md](./cloud-platforms.md) |
| Secrets management & rotation | [hashicorp-suite.md](./hashicorp-suite.md) (Vault), [cloud-platforms.md](./cloud-platforms.md) (Secrets Manager) |
| Service discovery in microservices | [hashicorp-suite.md](./hashicorp-suite.md) (Consul), [web-servers-and-proxies.md](./web-servers-and-proxies.md) (Traefik) |
| HTTP traffic ingress & routing | [web-servers-and-proxies.md](./web-servers-and-proxies.md) |
| TCP load balancing (databases, Redis, gRPC) | [web-servers-and-proxies.md](./web-servers-and-proxies.md) (HAProxy, Nginx stream) |
| Automatic HTTPS / TLS certificate management | [web-servers-and-proxies.md](./web-servers-and-proxies.md) (Caddy, Traefik), [hashicorp-suite.md](./hashicorp-suite.md) (Vault PKI) |
| Caching (sub-millisecond reads) | [redis.md](./redis.md) |
| Session storage (stateless app scaling) | [redis.md](./redis.md) |
| Rate limiting | [redis.md](./redis.md), [web-servers-and-proxies.md](./web-servers-and-proxies.md) (Nginx rate limit) |
| Leaderboards / real-time rankings | [redis.md](./redis.md) (Sorted Sets) |
| Distributed locks | [redis.md](./redis.md) (Redlock) |
| Async task queues & background jobs | [message-brokers.md](./message-brokers.md) (RabbitMQ, SQS) |
| Event streaming & replay | [message-brokers.md](./message-brokers.md) (Kafka, Pulsar), [redis.md](./redis.md) (Streams) |
| Change Data Capture (CDC) | [message-brokers.md](./message-brokers.md) (Kafka + Debezium) |
| Fan-out / pub-sub to multiple consumers | [message-brokers.md](./message-brokers.md) (Kafka, SNS+SQS, RabbitMQ fanout) |
| Workload orchestration (containers + VMs) | [hashicorp-suite.md](./hashicorp-suite.md) (Nomad), [cloud-platforms.md](./cloud-platforms.md) (EKS/GKE/AKS) |
| Immutable VM image building | [hashicorp-suite.md](./hashicorp-suite.md) (Packer) |
| Reproducible developer environments | [hashicorp-suite.md](./hashicorp-suite.md) (Vagrant), Dev Containers |
| Multi-cloud / hybrid infrastructure | [cloud-platforms.md](./cloud-platforms.md), [hashicorp-suite.md](./hashicorp-suite.md) (Terraform, Consul, Vault) |
| IoT / edge messaging | [message-brokers.md](./message-brokers.md) (NATS, MQTT via RabbitMQ) |
| Data pipeline / analytics ingestion | [message-brokers.md](./message-brokers.md) (Kafka Connect), [cloud-platforms.md](./cloud-platforms.md) (BigQuery, Kinesis) |

---

## Further Reading

### Within This Wiki
- **Distributed Systems** — For the theoretical underpinnings behind message brokers, consensus, and distributed state: see `../distributed-systems/`
  - [`../distributed-systems/consensus.md`](../distributed-systems/consensus.md) — Raft consensus (used by Vault, Consul, Kafka KRaft)
  - [`../distributed-systems/consistency-models.md`](../distributed-systems/consistency-models.md) — relevant when choosing between Redis Sentinel, Redis Cluster, and Kafka delivery guarantees
  - [`../distributed-systems/sagas.md`](../distributed-systems/sagas.md) — distributed transaction patterns across microservices using Kafka or RabbitMQ
  - [`../distributed-systems/outbox-pattern.md`](../distributed-systems/outbox-pattern.md) — reliable event publishing with Kafka/RabbitMQ from a transactional database
- **Databases** — For choosing datastores that pair with the infrastructure tools above: see `../database/`
  - [`../database/key-value-store.md`](../database/key-value-store.md) — deep dive on KV store design, relevant to Redis architecture
  - [`../database/nosql-databases.md`](../database/nosql-databases.md) — NoSQL options including DynamoDB, Bigtable, CosmosDB

### External References
- [HashiCorp Learn](https://developer.hashicorp.com/) — Official tutorials for Terraform, Vault, Consul, Nomad
- [AWS Architecture Center](https://aws.amazon.com/architecture/) — Reference architectures for AWS
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework) — GCP best practices
- [Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/) — Azure reference architectures
- [Redis Documentation](https://redis.io/docs/) — Official Redis reference
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/) — Official Kafka reference
- [Nginx Documentation](https://nginx.org/en/docs/) — Official Nginx reference
- [CNCF Landscape](https://landscape.cncf.io/) — Comprehensive map of cloud-native infrastructure tools

---

> **Note on Tool Selection Philosophy:** The best infrastructure stack is not the most sophisticated one — it is the simplest one that meets your current and near-future requirements. Over-engineering infrastructure at the startup stage creates operational debt that slows product velocity. Under-engineering at the enterprise stage creates reliability and security risks. Use this guide to match tool complexity to organisational maturity.
