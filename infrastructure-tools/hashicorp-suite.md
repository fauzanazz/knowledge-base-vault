# HashiCorp Suite — Comprehensive Reference

> **Last Updated:** April 2026  
> **Audience:** Platform engineers, DevOps / SRE practitioners, architects  
> **Scope:** Full HashiCorp ecosystem — OSS vs Enterprise, adoption guidance, comparisons

---

## Table of Contents

1. [Introduction & Philosophy](#introduction--philosophy)
2. [Terraform — Infrastructure as Code](#terraform--infrastructure-as-code)
3. [Vault — Secrets & Encryption](#vault--secrets--encryption)
4. [Consul — Service Discovery & Mesh](#consul--service-discovery--mesh)
5. [Nomad — Workload Orchestration](#nomad--workload-orchestration)
6. [Packer — Machine Image Automation](#packer--machine-image-automation)
7. [Vagrant — Developer Environments](#vagrant--developer-environments)
8. [Waypoint — Application Delivery](#waypoint--application-delivery)
9. [Tool Integration Patterns](#tool-integration-patterns)
10. [OSS vs Enterprise Decision Guide](#oss-vs-enterprise-decision-guide)
11. [Adoption Roadmap](#adoption-roadmap)
12. [Quick Reference](#quick-reference)

---

## Introduction & Philosophy

HashiCorp built its product portfolio around a single cohesive principle: **infrastructure should be treated as code, and every layer of that infrastructure should be addressable through a consistent, human-readable API.** Rather than shipping one monolithic platform, HashiCorp created purpose-built tools that each solve a discrete problem extremely well, yet integrate tightly when used together.

The company's founding thesis (articulated in the **Tao of HashiCorp**) centres on four pillars:
- **Workflows, not technologies** — tools should fit human workflows rather than forcing teams to adapt.
- **Simple, modular, composable** — each tool has a small surface area and integrates through well-defined contracts.
- **Communicating sequentially** — tools exchange structured data (HCL, JSON, APIs) rather than side-effects.
- **Developer experience first** — the local development loop must be fast and consistent with production.

**HashiCorp's 2023 BSL licence change** (from MPL-2.0 to BUSL-1.1) is critical context for any adoption decision. Tools from Terraform 1.6+, Vault 1.15+, Consul 1.17+, and Nomad 1.7+ are no longer OSI-approved open source for **competing use cases**. Pure internal infrastructure use is still free. The OpenTofu fork (Linux Foundation) maintains the original MPL-2.0 Terraform codebase and is gaining momentum.

---

## Terraform — Infrastructure as Code

### What It Does

Terraform is a **declarative, multi-cloud Infrastructure-as-Code (IaC) tool** that lets you define the desired state of cloud and on-premises infrastructure in **HashiCorp Configuration Language (HCL)** or JSON. An execution engine diffs desired state against the current state held in a *state file* and produces a **plan** of API calls to converge reality to intent.

### HCL Deep Dive

HCL is more than YAML with types. Key features:

```hcl
# Variables with validation
variable "environment" {
  type        = string
  description = "Deployment environment"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

# Dynamic blocks
resource "aws_security_group" "app" {
  name = "app-sg-${var.environment}"

  dynamic "ingress" {
    for_each = var.ingress_rules
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}

# Locals and expressions
locals {
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
    Team        = var.team_name
  }
}
```

**HCL advantages over YAML/JSON:**
- Native expression language (functions, for expressions, conditionals)
- First-class type system including objects, tuples, and sets
- Comments supported natively
- Designed to be machine-writable and human-readable simultaneously

### State Management

State is Terraform's memory — it maps real infrastructure resources to your configuration. State management is the most operationally complex part of Terraform at scale.

**Local state** — stored in `terraform.tfstate`. Fine for experimentation; catastrophic for teams (no locking, no history, no sharing).

**Remote state backends:**

| Backend | Locking | Encryption | Notes |
|---------|---------|-----------|-------|
| S3 + DynamoDB | ✅ DynamoDB | ✅ SSE | Most common AWS pattern |
| GCS | ✅ Native | ✅ CMEK | Best for GCP-primary shops |
| Azure Blob | ✅ Lease | ✅ BYOK | Best for Azure-primary shops |
| Terraform Cloud / HCP Terraform | ✅ Native | ✅ | SaaS, also stores plan history |
| PostgreSQL | ✅ Native | Depends | Self-hosted, full control |

**State isolation strategies:**
- **Workspaces** — single backend, multiple state files. Good for per-environment isolation in small organisations. Limits blast radius but shares backend infrastructure.
- **Directory-per-environment** — separate root modules per environment. More boilerplate but cleanest isolation.
- **Terragrunt** — a thin wrapper that adds DRY patterns, dependency management, and automated remote-state wiring across large mono-repos.

**State locking** prevents concurrent `apply` operations from corrupting state. Always use a locking-capable backend in production.

### Terraform vs Pulumi vs CloudFormation vs CDK

| Dimension | Terraform / OpenTofu | Pulumi | AWS CloudFormation | AWS CDK |
|-----------|----------------------|--------|-------------------|---------|
| **Language** | HCL (declarative) | Python, TS, Go, .NET, Java | YAML / JSON | TS, Python, Java, Go |
| **Multi-cloud** | ✅ 3,000+ providers | ✅ Same providers as Terraform | ❌ AWS only | ❌ AWS only (CDKtf = multi-cloud) |
| **State** | External (backend) | Pulumi Cloud or self-hosted | CloudFormation stacks (AWS managed) | CloudFormation (via synth) |
| **Execution model** | Plan/Apply | Preview/Up | Change sets | Synth → CFN deploy |
| **Loops/conditionals** | HCL meta-args (`count`, `for_each`) | Full language features | Limited (via macros) | Full language features |
| **Drift detection** | `terraform plan` | `pulumi refresh` | Drift detection (optional) | Via CFN |
| **Maturity** | Highest ecosystem | Growing fast | Most AWS-native | Rapidly growing |
| **Learning curve** | Low-medium | High (requires proficiency in chosen language) | Medium | Medium-high |
| **Secrets handling** | Mark as sensitive; not encrypted in state by default | Pulumi ESC / encrypted secrets | SSM Parameter Store / Secrets Manager | Via CDK constructs |

**When to choose Terraform/OpenTofu:**
- Multi-cloud or hybrid-cloud environments
- Large existing Terraform ecosystem or provider ecosystem
- Teams who prefer a purpose-built DSL over general-purpose languages
- When GitOps workflows and plan-review are culturally important

**When to choose Pulumi:**
- Strongly typed languages reduce config errors at scale
- Need complex logic (loops, conditionals, real unit tests) that HCL struggles with
- Already investing in TypeScript/Python as primary infra languages

**When to choose CloudFormation / CDK:**
- AWS-only shop needing deepest AWS feature parity (CFN often gets new features before Terraform providers)
- CDK constructs provide L2/L3 abstractions that encode best practices
- Avoiding external state backend management

### Key Best Practices

1. **Module structure** — keep modules small and reusable. One module = one logical unit (e.g., `vpc`, `rds-cluster`, `ecs-service`).
2. **Version pin providers** — always use `required_providers` with `~>` constraints to prevent surprise upgrades.
3. **Separate state by lifecycle** — networking state changes rarely; application state changes frequently. Split them.
4. **`terraform plan` in CI** — treat plan output as an artefact. Review before merge, apply after merge.
5. **Secrets** — never put secrets in HCL. Use `data` sources from Vault or cloud secret managers; mark outputs `sensitive = true`.

---

## Vault — Secrets & Encryption

### What It Does

Vault is a **secrets management, identity-based access, and data protection platform**. Its core value proposition goes beyond storing API keys: Vault can *generate* secrets dynamically and revoke them automatically, making every credential short-lived and auditable.

### Core Concepts

**Secrets Engines** — pluggable backends that generate or store secrets:
- **KV v2** — versioned key-value store for static secrets with full audit trail
- **AWS / GCP / Azure** — generates cloud provider credentials on-demand (IAM roles, service accounts) that expire automatically
- **Database** — creates short-lived database credentials for PostgreSQL, MySQL, MongoDB, Cassandra, and more
- **PKI** — full internal certificate authority; issues TLS certs with 5-minute TTLs if desired
- **Transit** — encryption-as-a-service; your application sends plaintext, gets ciphertext back, never manages keys

**Auth Methods** — how clients prove identity:
- Kubernetes (pod's service account JWT)
- AWS IAM (instance profile / assumed role)
- OIDC / JWT (integrate with your IdP — Okta, Google, GitHub OIDC)
- AppRole (CI/CD pipelines)
- LDAP / Active Directory

**Policies** — HCL-based ACL policies control what a given identity can read/write:

```hcl
# Policy: allow app to read its own DB credentials
path "database/creds/my-app-role" {
  capabilities = ["read"]
}

path "secret/data/my-app/*" {
  capabilities = ["read", "list"]
}
```

### Dynamic Secrets — The Killer Feature

Static secrets are inherently dangerous: they don't rotate, they get copied, they get committed to source control. Vault's dynamic secrets solve this at the source.

**Database example flow:**
1. App authenticates to Vault with Kubernetes service account JWT
2. Vault validates the JWT with the Kubernetes API
3. App requests `database/creds/my-app-role`
4. Vault creates a new PostgreSQL user with a 1-hour TTL
5. App receives username/password; uses it for the session
6. Vault revokes the user after TTL expires (or on app shutdown via `vault lease revoke`)

No long-lived credentials ever leave Vault. Every access is logged in the audit log.

### Encryption as a Service (Transit Engine)

Instead of managing keys in your application, the Transit engine provides an API:

```bash
# Encrypt
vault write transit/encrypt/my-key plaintext=$(base64 <<< "sensitive data")
# → ciphertext: vault:v1:abc123...

# Decrypt
vault write transit/decrypt/my-key ciphertext="vault:v1:abc123..."
# → plaintext: c2Vuc2l0aXZlIGRhdGE=
```

Benefits: key rotation happens in Vault without re-encrypting data (keys are versioned), FIPS 140-2 compliant HSM support in Enterprise, full audit trail of every encryption/decryption operation.

### Vault vs AWS Secrets Manager

| Dimension | HashiCorp Vault | AWS Secrets Manager |
|-----------|----------------|---------------------|
| **Hosting** | Self-managed or HCP Vault | Fully managed |
| **Multi-cloud** | ✅ Any cloud, on-prem | ❌ AWS-native (can store cross-cloud secrets) |
| **Dynamic secrets** | ✅ (databases, cloud, PKI, SSH) | ⚠️ Lambda rotation (custom, complex) |
| **Encryption as a service** | ✅ Transit engine | ⚠️ KMS (separate service) |
| **Auth methods** | 20+ (Kubernetes, OIDC, AWS, GCP, LDAP…) | AWS IAM only |
| **Audit logging** | Built-in, pluggable | CloudTrail |
| **Cost** | OSS: infra cost only; Enterprise: licence | $0.40/secret/month + $0.05/10k API calls |
| **Operational overhead** | High (HA cluster, unsealing, DR) | Zero |
| **Secret rotation** | Native for DB, PKI, cloud | Lambda-based rotation scripts |

**Choose Vault when:** multi-cloud or hybrid, need dynamic secrets beyond what Secrets Manager rotation supports, need encryption-as-a-service, need rich auth methods for Kubernetes or on-prem workloads.

**Choose Secrets Manager when:** AWS-only, small team, want zero operational burden, happy with Lambda-based rotation.

### High Availability & Operations

Production Vault requires:
- **Integrated Storage (Raft)** — built-in consensus, no external Consul dependency (recommended since Vault 1.4)
- **Unseal mechanism** — Shamir's Secret Sharing (multi-key), or Auto Unseal via cloud KMS (AWS KMS, GCP CKMS, Azure Key Vault) — always use Auto Unseal in production
- **3 or 5 node cluster** — one active, rest standby; leader election via Raft
- **Disaster Recovery (Enterprise)** — cross-region DR replication with near-zero RPO

---

## Consul — Service Discovery & Mesh

### What It Does

Consul is a **distributed service networking platform** providing service discovery, health checking, key-value storage, and a full service mesh (Consul Connect). It was originally designed as the service catalogue that would make microservices discoverable to each other without hardcoded IPs.

### Service Discovery

Services register with Consul (via agent, Kubernetes sync, or Terraform Consul provider). Clients query via DNS or HTTP API:

```bash
# DNS query — returns healthy instances only
dig @127.0.0.1 -p 8600 my-api.service.consul

# HTTP API
curl http://localhost:8500/v1/health/service/my-api?passing=true
```

**Health checking:** Consul supports HTTP, TCP, gRPC, TTL, and custom script checks. Unhealthy instances are automatically removed from DNS results, providing automatic failover without load balancer reconfiguration.

### Service Mesh (Consul Connect)

Consul Connect is a **Layer 4/7 service mesh** using Envoy proxies as sidecars. It provides:
- **mTLS between services** — no code changes; the proxy handles TLS
- **Intentions** — L4 allow/deny rules between services (`my-api` may talk to `my-db`, deny all others)
- **L7 traffic management** — weighted routing, circuit breakers, retries (via Envoy config)
- **Observability** — metrics and traces forwarded from Envoy to your observability stack

This positions Consul as a competitor/complement to Istio in the service mesh space. Consul's advantage is tighter integration with non-Kubernetes workloads (VMs, Nomad, bare metal).

### Consul vs etcd vs ZooKeeper

| Dimension | Consul | etcd | ZooKeeper |
|-----------|--------|------|-----------|
| **Primary use case** | Service discovery + mesh + KV | Distributed config / K8s backbone | Distributed coordination |
| **Service discovery** | ✅ First-class | ❌ (via external tooling) | ❌ (custom watches) |
| **Health checking** | ✅ Built-in | ❌ | ❌ |
| **Service mesh** | ✅ Connect (Envoy) | ❌ | ❌ |
| **KV store** | ✅ (simple, not ACID) | ✅ (strongly consistent, MVCC) | ✅ (ZNodes, watches) |
| **Consensus** | Raft | Raft | Zab |
| **Multi-datacenter** | ✅ First-class | ❌ (federation is complex) | ❌ |
| **Kubernetes native** | ⚠️ (via sync) | ✅ (K8s uses etcd) | ❌ |
| **Operational complexity** | Medium | Low (if using K8s) | High |

**Choose Consul when:** heterogeneous environment (VMs + containers + Kubernetes), need service mesh for non-K8s workloads, need multi-datacenter service registry.  
**Choose etcd when:** Kubernetes-native stack, need strongly consistent distributed locks, don't need service discovery features.  
**Choose ZooKeeper when:** running Kafka or other ZK-dependent software — otherwise, prefer newer alternatives.

---

## Nomad — Workload Orchestration

### What It Does

Nomad is a **workload orchestrator** that schedules and manages applications across a fleet of machines. Unlike Kubernetes, Nomad is intentionally minimal — it does one thing (scheduling) and relies on Consul and Vault for service discovery and secrets.

### Nomad vs Kubernetes

| Dimension | Nomad | Kubernetes |
|-----------|-------|------------|
| **Operational complexity** | Low | Very high |
| **Learning curve** | Low | Steep (significant abstraction layers) |
| **Workload types** | Containers, VMs (QEMU), Java JARs, binaries, batch | Primarily containers |
| **Scheduling model** | Bin-packing + spread | Bin-packing with extensive plugin ecosystem |
| **Cluster size** | Up to 10,000+ nodes tested | Practical limit ~5,000 nodes per cluster |
| **Non-container support** | ✅ First-class | ❌ (container-centric) |
| **Service mesh** | Via Consul Connect | Istio, Linkerd, Cilium |
| **GPU support** | ✅ | ✅ |
| **Windows workloads** | ✅ | ✅ (improving) |
| **Ecosystem/community** | Smaller | Massive |
| **Multi-region** | ✅ Federated clusters | Complex (Cluster API, KubeFed) |

**When to choose Nomad:**
- Small-to-medium teams without Kubernetes expertise
- Mixed workloads: containers AND VMs AND standalone binaries (legacy applications, batch jobs)
- Existing HashiCorp investment (Consul + Vault + Nomad = tight integration)
- Batch/data workloads where Kubernetes overhead is unjustified
- Edge computing or resource-constrained environments

**When to choose Kubernetes:**
- Container-only workloads at large scale
- Need the broadest ecosystem (Helm, operators, CNCF tools)
- Cloud-managed control plane (EKS, GKE, AKS) eliminates operational burden
- Team already proficient in Kubernetes

**Nomad job file example:**

```hcl
job "web-api" {
  datacenters = ["us-east-1"]
  type        = "service"

  group "api" {
    count = 3

    network {
      port "http" { to = 8080 }
    }

    task "api" {
      driver = "docker"
      config {
        image = "my-org/web-api:v1.2.3"
        ports = ["http"]
      }

      template {
        data        = <<EOF
{{- with secret "database/creds/api-role" }}
DB_USER={{ .Data.username }}
DB_PASS={{ .Data.password }}
{{- end }}
EOF
        destination = "secrets/db.env"
        env         = true
      }

      resources {
        cpu    = 500
        memory = 512
      }
    }

    service {
      name     = "web-api"
      port     = "http"
      provider = "consul"

      check {
        type     = "http"
        path     = "/health"
        interval = "10s"
        timeout  = "2s"
      }
    }
  }
}
```

---

## Packer — Machine Image Automation

### What It Does

Packer automates the creation of **identical machine images** across multiple platforms from a single source configuration. Instead of "configuration at boot" (cloud-init, Ansible playbooks that run every launch), Packer bakes configuration into the image — resulting in faster, more predictable instance launches.

### Immutable Infrastructure Pattern

Packer is the cornerstone of **immutable infrastructure**: rather than patching running servers, you build a new image with the patch applied, blue-green-deploy new instances, and terminate the old ones. This eliminates configuration drift and "works on my machine" issues.

```hcl
# packer build configuration
source "amazon-ebs" "ubuntu" {
  ami_name      = "my-app-${formatdate("YYYY-MM-DD", timestamp())}"
  instance_type = "t3.medium"
  region        = "us-east-1"
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/hvm-ssd/ubuntu-22.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    owners      = ["099720109477"] # Canonical
    most_recent = true
  }
  ssh_username = "ubuntu"
}

build {
  sources = ["source.amazon-ebs.ubuntu"]

  provisioner "ansible" {
    playbook_file = "./playbooks/app.yml"
  }

  post-processor "manifest" {
    output     = "manifest.json"
    strip_path = true
  }
}
```

Packer supports 20+ builders: AWS AMI, GCP Machine Image, Azure Managed Image, VMware OVF, Docker, VirtualBox, QEMU, and more — from one config.

**CI/CD integration:** Packer runs in CI, produces an AMI ID, which is stored in SSM Parameter Store. Terraform reads the parameter to deploy EC2 instances using the latest tested image.

---

## Vagrant — Developer Environments

### What It Does

Vagrant provides **reproducible, portable development environments** using a simple `Vagrantfile`. It manages virtual machines (VirtualBox, VMware, Hyper-V, libvirt) or containers (Docker) and provisions them consistently across developer machines.

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.network "forwarded_port", guest: 8080, host: 8080

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus   = 2
  end
end
```

### Current Relevance

Vagrant's role has significantly diminished with the rise of **Dev Containers** (VS Code + Docker), **GitHub Codespaces**, and **Nix/devenv**. However, Vagrant remains relevant when:
- Developers need **full VM isolation** (kernel-level testing, different OS, driver testing)
- Simulating multi-machine network topologies locally (e.g., Consul cluster on laptop)
- Teams on Windows needing Linux development environments without WSL complexity
- Testing Packer configurations before publishing

For most greenfield projects, Dev Containers or Codespaces are preferred over Vagrant.

---

## Waypoint — Application Delivery

### What It Does

Waypoint provides a **unified deployment workflow** — `waypoint build`, `waypoint deploy`, `waypoint release` — abstracting over underlying platforms (Kubernetes, Nomad, AWS ECS, Lambda, Docker, etc.). Think of it as a PaaS abstraction layer over your existing infrastructure.

```hcl
app "web" {
  build {
    use "pack" {}   # Cloud Native Buildpacks — no Dockerfile needed
    registry {
      use "docker" {
        image = "my-org/web"
        tag   = gitrefpretty()
      }
    }
  }

  deploy {
    use "kubernetes" {
      replicas = 3
      probe_path = "/health"
    }
  }

  release {
    use "kubernetes-ingress" {}
  }
}
```

**Current status:** Waypoint is one of HashiCorp's less mature products and has seen slower adoption than Terraform or Vault. HashiCorp has indicated reduced investment in Waypoint. Consider carefully before adopting; alternatives like **Backstage** (portal) + Argo CD (GitOps) or **Dagger** (pipeline as code) may be more future-proof for the application delivery abstraction layer.

---

## Tool Integration Patterns

### The Full HashiCorp Stack

```
Developer commits code
        │
        ▼
  Packer builds AMI / container image
  (baked configuration, Vault agent pre-installed)
        │
        ▼
  Terraform provisions infrastructure
  (VPC, ASG using Packer AMI, Consul cluster, Vault cluster, Nomad cluster)
        │
        ▼
  Consul registers services, health checks, provides DNS
        │
        ▼
  Vault issues dynamic credentials to workloads
  (Nomad jobs get Vault tokens via Nomad's Vault integration)
        │
        ▼
  Nomad schedules and runs workloads
  (jobs declare Vault secrets via template stanza)
        │
        ▼
  Consul Connect provides mTLS between services
```

### Vault + Kubernetes Integration

1. Enable Kubernetes auth method in Vault
2. Deploy Vault Agent Injector (mutating webhook) in cluster
3. Annotate pods with `vault.hashicorp.com/agent-inject: "true"` and desired secrets
4. Agent injector injects an init container + sidecar that handles auth and secret delivery
5. Secrets appear as files in `/vault/secrets/` — zero code changes in the application

### Terraform + Vault: Chicken-and-Egg

Bootstrap order matters:
1. Terraform provisions the Vault cluster (using cloud provider resources)
2. Vault is manually initialised and unsealed (or via Terraform Vault provider after the fact)
3. Terraform (in subsequent runs, authenticated to Vault) reads secrets from Vault for downstream resources

Use separate Terraform state for the Vault cluster itself to avoid circular dependencies.

---

## OSS vs Enterprise Decision Guide

| Feature | OSS / Community | Enterprise |
|---------|-----------------|------------|
| **Terraform** | Core IaC, all providers | Sentinel policy-as-code, audit logging, SSO, private registry, team management |
| **Vault** | All secrets engines, OSS HA | HSM support, DR replication, namespace multi-tenancy, MFA enforcement, KMIP |
| **Consul** | Service discovery, KV, Connect | Audit logging, enhanced federation, network segments, redundancy zones |
| **Nomad** | Full scheduler | Multi-region federation, SSO, Sentinel, resource quotas, audit logging |

**Enterprise is worth considering when:**
- Compliance requirements mandate audit logging (SOC 2, PCI-DSS, HIPAA)
- Multi-team environments need namespace isolation (Vault namespaces, Nomad quotas)
- Sentinel policy enforcement is required for compliance guardrails on Terraform runs
- DR replication with near-zero RPO is a business requirement

**HCP (HashiCorp Cloud Platform):** Managed versions of Vault, Consul, and Terraform (HCP Terraform, formerly Terraform Cloud) eliminate operational overhead. Good middle ground between OSS self-managed and full Enterprise licencing.

---

## Adoption Roadmap

### Stage 1 — Foundation (Months 1–3)
- Adopt **Terraform** for all new infrastructure (no more clicking in consoles)
- Set up remote state backend (S3 + DynamoDB or HCP Terraform)
- Enforce `terraform plan` in CI; require approval before `apply`

### Stage 2 — Secrets Hygiene (Months 3–6)
- Deploy **Vault** cluster with Auto Unseal via cloud KMS
- Migrate database credentials to dynamic secrets
- Integrate Vault with Kubernetes or your compute platform
- Eliminate `.env` files and hardcoded secrets from source control

### Stage 3 — Service Networking (Months 6–12)
- Deploy **Consul** for service discovery if operating in multi-platform or multi-cloud environments
- Enable Consul Connect for mTLS between critical services
- Replace hardcoded service addresses with DNS-based discovery

### Stage 4 — Image & Environment Consistency (Ongoing)
- Adopt **Packer** to build hardened base images; integrate with Terraform
- Consider **Vagrant** for complex local multi-machine development setups
- Evaluate **Nomad** if running non-container workloads alongside containers

---

## Quick Reference

| Tool | Problem Solved | Key Competitor | BSL Impact |
|------|--------------|---------------|------------|
| Terraform | Multi-cloud IaC | Pulumi, CloudFormation, CDK | OpenTofu fork for true OSS |
| Vault | Secrets management, encryption | AWS Secrets Manager, CyberArk | Internal use still free |
| Consul | Service discovery, mesh | etcd, Istio, AWS Cloud Map | Internal use still free |
| Nomad | Workload orchestration | Kubernetes, AWS ECS | Internal use still free |
| Packer | Machine image automation | EC2 Image Builder, Buildah | Still MPL (check latest) |
| Vagrant | Dev environment virtualisation | Dev Containers, Codespaces | Low adoption risk |
| Waypoint | App delivery abstraction | Heroku, Argo CD, Dagger | Reduced investment; evaluate carefully |

---

*For related topics, see: [Cloud Platforms Comparison](./cloud-platforms.md) | [Kubernetes Deep Dive](../orchestration/kubernetes.md) | [CI/CD Pipelines](../cicd/pipeline-patterns.md)*
