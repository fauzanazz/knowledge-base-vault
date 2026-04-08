# Cloud Platforms — AWS vs GCP vs Azure Comprehensive Comparison

> **Last Updated:** April 2026  
> **Audience:** Architects, platform engineers, engineering managers making cloud strategy decisions  
> **Scope:** Compute, storage, databases, networking, managed Kubernetes, pricing, and selection guidance

---

## Table of Contents

1. [Market Overview & Positioning](#market-overview--positioning)
2. [Compute — VMs & Serverless](#compute--vms--serverless)
3. [Object Storage](#object-storage)
4. [Databases — Relational](#databases--relational)
5. [Databases — NoSQL & NewSQL](#databases--nosql--newscal)
6. [Networking](#networking)
7. [Managed Kubernetes](#managed-kubernetes)
8. [Identity & Security](#identity--security)
9. [Observability & Monitoring](#observability--monitoring)
10. [Pricing Models & Cost Optimisation](#pricing-models--cost-optimisation)
11. [When to Choose Which Provider](#when-to-choose-which-provider)
12. [Multi-Cloud Strategy](#multi-cloud-strategy)
13. [Quick Reference Matrix](#quick-reference-matrix)

---

## Market Overview & Positioning

As of 2026, three hyperscalers dominate the cloud infrastructure market:

| Provider | Market Share (approx.) | Revenue (2025 est.) | Core Strength |
|----------|----------------------|---------------------|---------------|
| **AWS** | ~31% | ~$110B ARR | Breadth, maturity, ecosystem |
| **Microsoft Azure** | ~24% | ~$85B ARR | Enterprise, Microsoft stack, hybrid |
| **Google Cloud (GCP)** | ~12% | ~$42B ARR | Data/ML, Kubernetes, network quality |

The remaining ~33% is split between Alibaba Cloud, Oracle Cloud, IBM Cloud, and a long tail of regional providers.

**Philosophical differences:**
- **AWS** is the *everything store* of cloud — it has the most services, highest feature depth, and the largest ecosystem. Its pricing and UX complexity is legendary; the right service exists, but finding it requires expertise.
- **Azure** is the *enterprise integration platform* — deeply embedded in Windows/Active Directory/Microsoft 365 environments. Hybrid connectivity (ExpressRoute, Azure Arc) is best-in-class. Azure's services often lag AWS by 12–24 months in maturity.
- **GCP** is the *engineering-first cloud* — Google's internal infrastructure externalised. It leads in managed Kubernetes (GKE is the reference implementation), BigQuery, Spanner, and AI/ML (Vertex AI, TPUs). Historically weak in enterprise support and sales; improving rapidly.

---

## Compute — VMs & Serverless

### Virtual Machines

#### AWS — EC2 (Elastic Compute Cloud)

EC2 is the most comprehensive VM service with **750+ instance types** across families:

| Family | Purpose | Examples |
|--------|---------|---------|
| General purpose | Balanced CPU/memory | `t4g`, `m7i`, `m7g` (Graviton) |
| Compute optimised | CPU-heavy workloads | `c7i`, `c7g`, `c7a` |
| Memory optimised | In-memory databases, caches | `r7i`, `x2gd`, `u-24tb1` |
| Storage optimised | IOPS-intensive | `i4i`, `d3en` |
| Accelerated computing | GPU/ML | `p4de` (A100), `trn1` (Trainium) |
| HPC | MPI, scientific | `hpc7g`, `hpc6a` |

**Graviton (ARM) instances** — AWS's custom ARM chips offer 20–40% better price/performance than x86 equivalents. `m7g`, `c7g`, `r7g` are the default choice for new workloads.

EC2 purchasing options:
- **On-Demand** — pay by the second; highest rate; no commitment
- **Reserved Instances (RI)** — 1 or 3-year commitment; up to 72% discount; Convertible RIs allow family/size changes
- **Savings Plans** — like RIs but flexible (Compute Savings Plans apply across EC2, Fargate, Lambda)
- **Spot Instances** — up to 90% discount; can be interrupted with 2-minute notice; excellent for batch, CI/CD, ML training
- **Dedicated Hosts / Instances** — compliance (BYOL, regulatory isolation)

#### GCP — Compute Engine (GCE)

GCP uses a simpler, more predictable instance taxonomy:

| Series | Purpose | Notes |
|--------|---------|-------|
| `n2`, `n2d` | General purpose | AMD EPYC for `n2d` |
| `c3` | Compute optimised | Latest generation |
| `m3` | Memory optimised | Up to 12TB RAM |
| `a2`, `a3` | GPU | NVIDIA A100, H100 |
| `t2d`, `t2a` | Cost-optimised (Tau) | AMD / Arm; 42% cheaper |

GCP's pricing differentiator: **Sustained Use Discounts (SUD)** — automatically applied when you run an instance for >25% of the month. No upfront commitment required. Combined with **Committed Use Discounts (CUD)** (1 or 3-year), GCP is often the most cost-competitive for steady-state workloads.

#### Azure — Virtual Machines

Azure VMs use a naming scheme reflecting generation, family, and capabilities:
- **B-series** — burstable, cheap dev/test
- **D-series** — general purpose (Dv5, Dsv5 for premium SSD)
- **F-series** — compute optimised
- **E-series** — memory optimised
- **N-series** — GPU (NCv3 = V100, NDm A100 v4 = A100)
- **HB / HC** — HPC

Azure **Spot VMs** and **Reserved VM Instances** mirror AWS equivalents. **Azure Hybrid Benefit** is a significant differentiator: if you have existing Windows Server or SQL Server licences with Software Assurance, you can apply them to Azure VMs for up to 49% savings — this is a major factor for enterprise Microsoft shops.

**Azure Arc** extends Azure management (RBAC, Policy, Defender) to on-premises and other-cloud VMs, making Azure the clear leader for hybrid scenarios.

### Serverless Functions

#### AWS Lambda

The original FaaS offering. Lambda supports Python, Node.js, Java, Go, .NET, Ruby, and custom runtimes. Key capabilities:
- **Cold starts:** 100ms–1s depending on runtime; SnapStart (Java) reduces to ~10ms
- **Concurrency:** 1,000 concurrent executions per region by default; soft limit increase available
- **Triggers:** 200+ event sources — API Gateway, S3, DynamoDB Streams, Kinesis, SQS, SNS, EventBridge
- **Max execution time:** 15 minutes
- **Memory:** 128MB–10GB; CPU scales proportionally
- **Pricing:** $0.20/million requests + $0.0000166667/GB-second; first 1M requests/month free
- **Deployment:** ZIP or container image (up to 10GB)

Lambda's ecosystem depth (Powertools for Lambda, extensive trigger integrations, Lambda Layers, Step Functions) is unmatched.

#### GCP Cloud Functions / Cloud Run

**Cloud Functions (2nd gen)** is built on Cloud Run under the hood. **Cloud Run** is Google's recommended serverless compute surface:
- Runs any container (not just functions) — significant flexibility advantage
- Scales to zero by default; first request cold start
- **Cloud Run Jobs** — for batch processing (vs Lambda, which is async/event-driven)
- Minimum instances configurable to eliminate cold starts
- Pricing: $0.00002400/vCPU-second + $0.00000250/GB-second; 180K vCPU-seconds free/month
- **Cloud Run for Anthos** — same experience on GKE, on-premises via Anthos

Cloud Run's container-native approach is often preferred by teams already containerising their applications.

#### Azure Functions

- Supports C#, JavaScript/TypeScript, Python, Java, PowerShell, Go (custom handler)
- **Durable Functions** — stateful orchestration patterns (fan-out, human approval, chaining) without a separate Step Functions equivalent
- Runs in Consumption Plan (serverless), Premium Plan (pre-warmed, VNet integration), or Dedicated Plan
- Triggers: HTTP, Timer, Blob Storage, Queue, CosmosDB, EventHub, Service Bus, and more
- Maximum execution time: 10 minutes (Consumption), unlimited (Premium/Dedicated)
- Deep integration with **Azure Logic Apps** for low-code orchestration

**Key differentiator:** Durable Functions makes complex stateful workflows much simpler than AWS Step Functions; Azure Functions Premium Plan offers always-warm instances without leaving the Functions paradigm.

---

## Object Storage

### AWS S3 (Simple Storage Service)

The *original* cloud service and still the gold standard. S3 defined object storage.

**Storage classes:**
| Class | Use Case | Retrieval | Price/GB/month |
|-------|---------|-----------|---------------|
| Standard | Frequently accessed | Immediate | $0.023 |
| Intelligent-Tiering | Unknown access pattern | Immediate | $0.023 + monitoring fee |
| Standard-IA | Infrequent access | Immediate | $0.0125 |
| One Zone-IA | Non-critical, infrequent | Immediate | $0.01 |
| Glacier Instant | Archive, quarterly access | Milliseconds | $0.004 |
| Glacier Flexible | Long-term archive | Minutes–hours | $0.0036 |
| Glacier Deep Archive | 7+ year retention | 12 hours | $0.00099 |

**Key features:** S3 Object Lock (WORM compliance), versioning, replication (CRR/SRR), Event Notifications, S3 Select (SQL queries on objects), Mountpoint for S3 (POSIX mount), Storage Lens (analytics), access points.

**S3 is effectively AWS's data lake foundation** — nearly every AWS service integrates with it natively.

### GCP Cloud Storage (GCS)

Near feature-parity with S3 but with GCP-native strengths:

| Class | Use Case | Price/GB/month |
|-------|---------|---------------|
| Standard | Frequently accessed | $0.020 |
| Nearline | Monthly access | $0.010 |
| Coldline | Quarterly access | $0.004 |
| Archive | Annual access | $0.0012 |

**GCS advantages:** No retrieval fees for Standard class (AWS charges per-request), single global namespace for multi-region buckets, direct BigQuery integration (external tables with zero egress cost within GCP), **strong consistency** (immediate, vs S3's eventual consistency for list operations prior to 2020). **Cloud Storage FUSE** for POSIX mount.

### Azure Blob Storage

Azure Blob uses a **storage account → container → blob** hierarchy. Three blob types:
- **Block blobs** — object storage equivalent (up to 190.7 TB)
- **Append blobs** — log data, sequential writes only
- **Page blobs** — random read/write, used for VM disks (managed disks use these under the hood)

Access tiers: Hot ($0.018/GB), Cool ($0.01/GB), Cold ($0.0045/GB), Archive ($0.00099/GB).

**Azure Data Lake Storage Gen2 (ADLS Gen2)** = Blob Storage + hierarchical namespace — enables POSIX ACLs and performant directory operations for big data workloads.

**Azure-unique feature:** Blob Storage is deeply integrated with **Azure CDN**, **Azure Static Web Apps**, and the Microsoft 365 ecosystem. Storage accounts also host Azure Files (SMB/NFS shares) and Azure Queues — converging multiple protocols in one account.

---

## Databases — Relational

### AWS RDS & Aurora

**RDS** manages PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, and Db2 with automated backups, Multi-AZ failover, and read replicas.

**Aurora** is AWS's cloud-native relational database — fully compatible with MySQL and PostgreSQL but redesigned storage layer:
- Storage auto-scales 10GB → 128TB
- 6-way replication across 3 AZs at the storage layer (not replication at the application level)
- **Aurora Serverless v2** — scales capacity in ~0.5 ACU increments; from 0.5 to 128 ACUs; excellent for variable workloads
- **Aurora Global Database** — cross-region replica with <1s replication lag, promotes in <1 minute
- Aurora I/O Optimised pricing — better for I/O-heavy workloads

**Aurora is typically the right choice** for new AWS-native relational workloads. Standard RDS is preferable for Oracle/SQL Server licencing, specific version requirements, or cost sensitivity on small databases.

### GCP Cloud SQL & AlloyDB

**Cloud SQL** manages PostgreSQL, MySQL, and SQL Server. Straightforward, well-integrated with GCP services, supports read replicas and automated failover.

**AlloyDB** is GCP's Aurora equivalent — PostgreSQL-compatible, columnar engine for analytics, 4× faster than standard PostgreSQL, 2× faster than Aurora for transaction-heavy workloads (per Google benchmarks). AlloyDB Omni can run on-premises or other clouds. Strong choice when on GCP and need Aurora-class performance.

### Azure SQL & Managed Instances

**Azure SQL Database** — Serverless tier, Hyperscale tier (similar to Aurora's storage layer, scales to 100TB), and General Purpose / Business Critical tiers. Strongest SQL Server compatibility including T-SQL, SQL Agent, linked servers.

**Azure SQL Managed Instance** — near 100% SQL Server engine compatibility (for lift-and-shift of legacy SQL Server workloads). Supports cross-database queries, SQL Agent, CLR, and other features SQL Database omits.

**Azure Database for PostgreSQL Flexible Server** — best PostgreSQL-as-a-service on Azure; zone-redundant HA.

---

## Databases — NoSQL & NewSQL

### AWS DynamoDB

DynamoDB is a **serverless, fully managed NoSQL database** — key-value and document model.

Architecture decisions:
- **Partition key** design is everything — poor partition key choices lead to hot partitions and throttling
- **On-Demand vs Provisioned capacity** — on-demand scales instantly but costs ~6× more per RCU/WCU vs well-provisioned
- **Global Tables** — multi-region, multi-master with ~1s replication lag
- **DynamoDB Streams** — change data capture for event-driven architectures
- **DAX (DynamoDB Accelerator)** — in-memory cache, sub-millisecond reads for read-heavy workloads
- **PartiQL** — SQL-compatible query language for DynamoDB (does not change underlying scalability properties)

**DynamoDB trade-offs:**
- Single-digit millisecond latency at any scale (genuinely exceptional)
- No joins; data modelling requires denormalisation
- Limited query patterns (primary key or GSI only)
- Expensive for write-heavy workloads at scale vs self-managed databases

### GCP Bigtable

Bigtable is a **wide-column, petabyte-scale NoSQL database** — the technology behind Google Search and Google Maps.

- Row key → column families → columns → timestamped values
- Ideal for **time-series**, **IoT telemetry**, **financial tick data**, **clickstream** data
- Scales to millions of rows per second, petabytes of storage
- Low latency: <10ms for row reads, <5ms for writes
- No server management; instances scale horizontally by adding nodes
- **Not a general-purpose database** — designed for specific access patterns at massive scale

Bigtable vs DynamoDB: Bigtable excels at append-heavy time-series at extreme scale. DynamoDB excels at transactional key-value / document access patterns with bursty traffic.

### Azure CosmosDB

CosmosDB is Azure's **globally distributed, multi-model database** — arguably the most feature-rich NoSQL offering from any cloud:

- **Multiple APIs**: Core (SQL/document), MongoDB, Cassandra, Gremlin (graph), Table
- **Five consistency levels**: Strong → Bounded Staleness → Session → Consistent Prefix → Eventual; tunable per-request
- **Multi-region writes** — write to any region, automatic conflict resolution
- **Autoscale** — scales RU/s automatically within configured range
- **Serverless** — for bursty, infrequent workloads
- **SLA**: 99.999% for multi-region, 99.99% for single-region

CosmosDB's flexibility (multi-model APIs, tunable consistency) makes it a strong choice when migrating legacy MongoDB, Cassandra, or HBase workloads to Azure without a full rewrite. The Cost can be high at scale — RU pricing requires careful capacity planning.

---

## Networking

### AWS Networking

**VPC (Virtual Private Cloud)** — the foundation. Subnets, route tables, Internet Gateway, NAT Gateway, VPC Peering, Transit Gateway.

**Transit Gateway** — hub-and-spoke connectivity for many VPCs across accounts and regions. Replaces the n² complexity of VPC peering at scale. Supports inter-region peering, VPN attachment, and Direct Connect Gateway attachment.

**Direct Connect** — dedicated physical connection to AWS. 1Gbps–100Gbps. Lower latency and more consistent throughput than VPN. Required for latency-sensitive or compliance-driven hybrid architectures.

**Route 53** — DNS + health checking + latency/geolocation/weighted routing. Resolver for hybrid DNS.

**CloudFront** — CDN with 450+ PoPs; integrates with WAF, Shield, Lambda@Edge.

**PrivateLink** — expose services to other VPCs/accounts without routing through the internet; SaaS vendors increasingly offer PrivateLink endpoints.

### GCP Networking

GCP's networking model is architecturally different: **VPCs are global** (subnets are regional, but a single VPC spans all regions by default). This eliminates much of the peering complexity seen in AWS.

**Cloud Interconnect** (Dedicated or Partner) — equivalent to Direct Connect.

**Cloud CDN** — tight integration with Cloud Load Balancing (all in one config).

**Premium vs Standard Network Tier:**
- **Premium** — routes traffic over Google's private backbone as long as possible (enters Google's network at the PoP closest to the sender)
- **Standard** — routes over public internet until reaching GCP; significantly cheaper but higher latency/variance

Premium Tier's backbone quality is a genuine competitive advantage for latency-sensitive global applications.

**VPC Service Controls** — perimeter-based security that restricts Google APIs to specific VPCs; mitigates data exfiltration via compromised credentials.

### Azure Networking

**Virtual Network (VNet)** — regional, similar to AWS VPC. VNet Peering for connectivity. **Virtual WAN** is the equivalent of AWS Transit Gateway for hub-and-spoke at scale.

**ExpressRoute** — equivalent to Direct Connect. **ExpressRoute Global Reach** connects on-premises locations to each other via Microsoft's backbone — unique capability without an AWS/GCP equivalent.

**Azure Front Door** — global HTTP load balancer + CDN + WAF in one service; routes based on latency and health checks.

**Azure Private Endpoint / Private Link** — same concept as AWS PrivateLink; widely adopted for PaaS services (SQL, Storage, Key Vault).

**Azure Firewall** — stateful, managed network firewall. **Network Security Groups (NSGs)** — security groups at subnet and NIC level.

**Hybrid advantage:** Azure's hybrid networking story (ExpressRoute with Global Reach, Azure Arc, Azure Stack HCI) is the most complete of the three providers for organisations with significant on-premises presence.

---

## Managed Kubernetes

### AWS EKS (Elastic Kubernetes Service)

EKS is a managed Kubernetes control plane. AWS manages etcd and the API server; you manage worker nodes (EC2, Fargate, or managed node groups).

**EKS key capabilities:**
- **EKS Managed Node Groups** — AWS handles node provisioning, patching, and replacement
- **EKS on Fargate** — serverless worker nodes; no EC2 management; per-pod billing
- **EKS Anywhere** — run EKS on-premises (bare metal, VMware)
- **EKS Add-ons** — managed CNI (VPC CNI), CoreDNS, kube-proxy, EBS CSI driver
- **Karpenter** — open-source node autoscaler (created by AWS) that provisions optimal EC2 instances in <60 seconds; far superior to Cluster Autoscaler for cost optimisation

**EKS trade-offs:**
- Control plane: $0.10/hour ($72/month) per cluster
- Node groups are well-integrated with EC2 features (Spot, placement groups, launch templates)
- Version management and upgrade experience historically cumbersome (improving)
- AWS-specific integrations (ALB Ingress Controller, EFS CSI, IAM Roles for Service Accounts) add AWS lock-in

### GCP GKE (Google Kubernetes Engine)

GKE is universally considered the **most mature, feature-complete managed Kubernetes** offering. Kubernetes was invented at Google; GKE ships features well before EKS or AKS.

**GKE key capabilities:**
- **Autopilot mode** — fully managed node provisioning; GKE selects optimal nodes for each pod; billing per pod CPU/memory; eliminates node management entirely. Strong default choice.
- **Standard mode** — manage your own node pools; maximum control
- **Workload Identity** — maps Kubernetes service accounts to GCP service accounts; most elegant solution to workload IAM of the three providers
- **Dataplane V2** (eBPF-based) — network policy enforcement with better observability
- **GKE Backup** — application-consistent backup for clusters and PVCs
- **Multi-cluster Ingress** — globally load balance across GKE clusters in multiple regions from a single VIP
- **GKE Enterprise (Anthos)** — unified fleet management across GKE, on-premises, other clouds

**Pricing:** No control plane cost in Autopilot (compute billed per pod). Standard mode: first cluster free (one zonal free), additional clusters $0.10/hour. Autopilot is often cheaper for variable workloads; Standard is cheaper for predictable, high-utilisation clusters.

**GKE is the reference implementation recommendation** for teams where Kubernetes is the primary workload platform and cloud isn't a fixed constraint.

### Azure AKS (Azure Kubernetes Service)

AKS is Azure's managed Kubernetes. Control plane is free (Azure does not charge for the control plane).

**AKS key capabilities:**
- **Azure CNI Overlay** — modern networking mode reducing IP exhaustion issues in large clusters
- **Workload Identity** — integrates with Azure AD Workload Identity (OIDC federation); matured significantly
- **KEDA (Kubernetes Event-Driven Autoscaling)** — open-source project created by Microsoft; scales pods based on external event sources (queue depth, Kafka lag, etc.)
- **Virtual Nodes** — burst workloads to Azure Container Instances (serverless nodes) without pre-provisioning
- **Azure Policy for AKS** — Gatekeeper-based policy enforcement integrated with Azure Policy
- **Windows node pools** — Windows container workloads alongside Linux
- **AKS Automatic** (Preview) — similar to GKE Autopilot; fully managed node lifecycle

**Free control plane** is a significant cost advantage for organisations running many small clusters (dev, staging, feature-branch environments).

**Comparison Summary:**

| Dimension | EKS | GKE | AKS |
|-----------|-----|-----|-----|
| **Kubernetes feature velocity** | Medium | Highest | Medium |
| **Node management** | Managed node groups + Karpenter | Autopilot / Standard | Managed node pools |
| **Control plane cost** | $0.10/hr | $0.10/hr (Standard), free (Autopilot) | Free |
| **Serverless nodes** | Fargate | GKE Autopilot | Virtual Nodes |
| **Workload identity** | IRSA | Workload Identity | Azure AD Workload Identity |
| **Multi-cluster** | EKS Anywhere + mesh | Multi-cluster Ingress + Anthos | Azure Arc |
| **Windows containers** | ✅ | ⚠️ (limited) | ✅ |
| **Best for** | AWS-native stacks | K8s-first; best features | Azure/enterprise shops |

---

## Identity & Security

### AWS IAM, GCP IAM, Azure RBAC

All three providers offer Role-Based Access Control, but the models differ significantly:

**AWS IAM:** Policy-based (JSON documents attached to identities or resources). Extremely granular but complex. **AWS Organizations** + Service Control Policies (SCPs) for multi-account governance. **IAM Identity Center** (formerly SSO) for human users.

**GCP IAM:** Simpler, resource hierarchy (Organisation → Folder → Project → Resource). Roles are granted at any level and inherited downward. **Workforce Identity Federation** for external IdP integration without sync.

**Azure RBAC:** Integrated with **Azure Active Directory (now Entra ID)**. Built-in roles + custom roles. **Management Groups** for hierarchy above subscriptions. Tightest enterprise directory integration of the three.

---

## Observability & Monitoring

| Capability | AWS | GCP | Azure |
|-----------|-----|-----|-------|
| **Metrics** | CloudWatch | Cloud Monitoring | Azure Monitor |
| **Logs** | CloudWatch Logs | Cloud Logging | Log Analytics (Azure Monitor) |
| **Traces** | X-Ray | Cloud Trace | Application Insights |
| **Dashboards** | CloudWatch Dashboards | Cloud Monitoring Dashboards | Azure Monitor Workbooks |
| **Alerts** | CloudWatch Alarms | Cloud Alerting | Azure Monitor Alerts |

Third-party tools (Datadog, Grafana Cloud, New Relic, Dynatrace) provide unified observability across all three clouds and are often preferred over native tools for multi-cloud environments.

---

## Pricing Models & Cost Optimisation

### Pricing Philosophy

- **AWS:** Granular per-service pricing; many free tiers. Complexity requires cost management tooling (Cost Explorer, Budgets, Trusted Advisor). Data egress pricing is a major pain point.
- **GCP:** Sustained Use Discounts automatically reduce bills for continuous workloads. Committed Use Discounts (1 or 3-year). Competitive data egress pricing. GCS no retrieval fees for Standard.
- **Azure:** Reserved Instances + Azure Hybrid Benefit for licence-heavy shops. Dev/Test pricing for non-production environments (significant discount). EA (Enterprise Agreement) unlocks large custom discounts.

### Data Egress Pricing (a key hidden cost)

All three providers charge for data leaving their network. Approximate rates (2026, US regions):

| Provider | First 10TB/month | Next 40TB | >150TB |
|----------|-----------------|-----------|--------|
| AWS | $0.09/GB | $0.085/GB | $0.07/GB |
| GCP (Premium) | $0.08/GB | $0.06/GB | $0.05/GB |
| Azure | $0.087/GB | $0.083/GB | $0.07/GB |

Intra-region, inter-AZ transfer is charged by all providers (AWS: $0.01/GB each way for inter-AZ). GCP does not charge for intra-region traffic within the same VPC. This can be a meaningful difference for highly distributed microservices.

### Cost Optimisation Strategies

**AWS:**
1. Graviton instances — 20–40% cheaper than x86 equivalents
2. Compute Savings Plans — most flexible commitment discount
3. Spot for batch/CI/ML training — up to 90% savings
4. S3 Intelligent-Tiering — automatic tiering; no retrieval surprises
5. RDS Reserved Instances for steady-state databases

**GCP:**
1. Sustained Use Discounts — automatic, no commitment required
2. Spot VMs for interruptible workloads — up to 91% discount
3. GKE Autopilot — only pay for pod resources, not node capacity
4. Standard Network Tier — cheaper egress for latency-tolerant workloads
5. BigQuery on-demand vs flat-rate — choose based on query frequency

**Azure:**
1. Azure Hybrid Benefit — largest savings for Windows/SQL Server shops
2. Reserved VMs — up to 72% off on-demand
3. Dev/Test subscriptions — up to 55% off for non-production
4. Spot VMs for batch workloads
5. Azure Advisor — automated recommendation engine

---

## When to Choose Which Provider

### Choose AWS When:
- **Breadth of services matters** — no other provider matches the number of specialised services (Kinesis, SageMaker, Step Functions, AppSync, etc.)
- **Startup ecosystem** — AWS credits, Activate programme, extensive startup tooling
- **Highest maturity** for most services — features tend to be most polished and well-documented
- **US government / regulated industries** — AWS GovCloud, FedRAMP High, ITAR, extensive compliance certifications
- **Hybrid via Outposts** — AWS Outposts brings EC2/RDS/EKS to on-premises racks (niche but growing)

### Choose GCP When:
- **Data and ML are primary workloads** — BigQuery, Vertex AI, Dataflow (Apache Beam), TPUs for training
- **Kubernetes-native architecture** — GKE is objectively the best managed K8s; Anthos for multi-cloud
- **Networking quality is critical** — Premium Tier's global backbone reduces latency for globally distributed users
- **Firebase integration** — mobile/web real-time data tier
- **Cost for steady-state compute** — SUDs often make GCP cheapest for always-on workloads
- **Google Workspace integration** — single-vendor for productivity + cloud

### Choose Azure When:
- **Microsoft-heavy enterprise** — Active Directory/Entra ID, Microsoft 365, Teams, Windows Server, SQL Server
- **Hybrid infrastructure** — Azure Arc, Azure Stack HCI, ExpressRoute Global Reach
- **Existing EA (Enterprise Agreement)** — licensing benefits, Azure Hybrid Benefit
- **Microsoft ISV ecosystem** — many enterprise SaaS products (SAP, Oracle, Dynamics 365) certify on Azure first
- **European market** — Azure has strong regulatory posture in EU (EU Data Boundary, GDPR, NIS2)
- **Developer platform (GitHub + Azure DevOps)** — unified ALM → cloud delivery pipeline

### Multi-Cloud Triggers:
- Regulatory data residency requirements in regions where not all providers are present
- M&A activity forcing integration of different cloud footprints
- Specific best-of-breed services (GCP BigQuery for analytics on AWS-primary architecture)
- Avoiding single-vendor dependency for critical workloads
- Shadow IT reduction through provider consolidation (consolidate to 2 providers max)

---

## Multi-Cloud Strategy

### Reality Check

Multi-cloud is **often an aspiration, rarely a success**. The complexity of managing IAM, networking, security posture, observability, and cost across two or more providers multiplies non-linearly. Most organisations running "multi-cloud" are actually doing **multi-cloud by accident** (different teams chose different providers) rather than by design.

**Genuine multi-cloud use cases:**
1. **Data sovereignty** — Primary in AWS US, EU data stays in Azure West Europe for GDPR
2. **Best-of-breed** — GCP BigQuery + Looker for analytics on an AWS primary workload
3. **Disaster recovery** — secondary region on a different provider (extremely rare; most use same-provider multi-region)
4. **Negotiation leverage** — having credible alternatives keeps pricing competitive during EA renewal

### Abstraction Layers for Multi-Cloud

| Layer | Tools |
|-------|-------|
| **IaC** | Terraform/OpenTofu (providers for all clouds), Pulumi |
| **Kubernetes** | Any managed K8s + Helm; GitOps with Argo CD |
| **Service Mesh** | Consul (Nomad+VMs), Istio, Cilium |
| **Secrets** | HashiCorp Vault (multi-cloud native) |
| **Observability** | Datadog, Grafana Cloud, OpenTelemetry |
| **Cost** | Apptio Cloudability, CloudHealth, Vantage |
| **Security posture** | Wiz, Prisma Cloud, CSPM tools |

---

## Quick Reference Matrix

| Category | AWS | GCP | Azure |
|----------|-----|-----|-------|
| **VM compute** | EC2 (Graviton ⭐) | Compute Engine (SUDs ⭐) | Virtual Machines (Hybrid Benefit ⭐) |
| **Serverless functions** | Lambda | Cloud Run / Cloud Functions | Azure Functions (Durable ⭐) |
| **Container platform** | ECS / Fargate | Cloud Run | Azure Container Apps |
| **Managed Kubernetes** | EKS + Karpenter | GKE (best ⭐) | AKS (free control plane ⭐) |
| **Object storage** | S3 (deepest features ⭐) | GCS (no retrieval fees ⭐) | Azure Blob (ADLS Gen2 ⭐) |
| **Managed SQL** | RDS / Aurora ⭐ | Cloud SQL / AlloyDB | Azure SQL / Managed Instance ⭐ |
| **NoSQL / KV** | DynamoDB ⭐ | Bigtable (extreme scale ⭐) | CosmosDB (multi-model ⭐) |
| **NewSQL / distributed** | Aurora Global | Spanner ⭐ | CosmosDB |
| **Data warehouse** | Redshift | BigQuery ⭐ | Azure Synapse |
| **Networking** | Transit Gateway ⭐ | Global VPC ⭐ | ExpressRoute Global Reach ⭐ |
| **CDN** | CloudFront | Cloud CDN | Azure Front Door |
| **DNS** | Route 53 ⭐ | Cloud DNS | Azure DNS |
| **IaaS hybrid** | Outposts | Anthos | Azure Arc ⭐ |
| **Identity** | IAM + Identity Center | Workforce Identity Federation | Entra ID ⭐ |
| **ML/AI platform** | SageMaker | Vertex AI + TPUs ⭐ | Azure AI / OpenAI Service ⭐ |
| **Market maturity** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Enterprise integration** | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **Data/ML strength** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

⭐ = Category leader or notable differentiator

---

*For related topics, see: [HashiCorp Suite](./hashicorp-suite.md) | [Kubernetes Deep Dive](../orchestration/kubernetes.md) | [Networking Patterns](../networking/networking-patterns.md) | [Cost Optimisation](../operations/cost-optimisation.md)*
