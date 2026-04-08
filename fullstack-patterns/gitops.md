---
title: "GitOps"
category: fullstack-patterns
summary: "An operational framework using Git as the single source of truth for declarative infrastructure and application state, with automated agents continuously reconciling desired state with actual state."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# GitOps

> An operational framework using Git as the single source of truth for declarative infrastructure and application state, with automated agents continuously reconciling desired state with actual state.

## Core Principles

1. **Declarative**: System state described in files (Kubernetes YAML, Helm charts, Kustomize)
2. **Versioned**: All state stored in Git — full audit trail, rollback = `git revert`
3. **Pulled automatically**: Agents pull changes from Git (not CI pushing to cluster)
4. **Continuously reconciled**: Agents detect and correct drift every 30–60s

The pull-based model is critical: clusters never need inbound network access from CI/CD. The agent runs inside the cluster and polls Git.

## ArgoCD

ArgoCD is the most widely adopted GitOps tool (CNCF graduated):

```yaml
# ArgoCD Application manifest
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-api
  namespace: argocd
spec:
  project: production
  source:
    repoURL: https://github.com/myorg/gitops-config
    targetRevision: main
    path: apps/my-api/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: my-api
  syncPolicy:
    automated:
      prune: true        # delete resources removed from git
      selfHeal: true     # revert manual kubectl changes
    syncOptions:
      - CreateNamespace=true
```

**App of Apps pattern**: one ArgoCD Application manages child Applications — scales to hundreds of services:
```yaml
# Root application bootstraps all others
source:
  path: apps/         # directory of Application manifests
```

**ApplicationSets**: template-driven apps for multi-cluster, multi-environment:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
spec:
  generators:
    - matrix:
        generators:
          - git:
              files:
                - path: "clusters/*/config.yaml"
          - list:
              elements:
                - env: staging
                - env: production
```

## Flux (GitOps Toolkit)

Flux v2 is the alternative CNCF GitOps solution, more modular than ArgoCD:

```yaml
# Flux GitRepository source
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: gitops-config
spec:
  interval: 1m
  url: https://github.com/myorg/gitops-config
  ref:
    branch: main

# Flux Kustomization (applies path in repo)
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: my-api
spec:
  interval: 5m
  path: "./apps/my-api"
  prune: true
  sourceRef:
    kind: GitRepository
    name: gitops-config
```

**Flux vs. ArgoCD**:
| | ArgoCD | Flux |
|--|--------|------|
| UI | Rich dashboard | Basic (Weave GitOps adds UI) |
| Multi-tenancy | AppProject isolation | Namespace isolation |
| Secret management | Bitnami Sealed Secrets, Vault | SOPS native integration |
| Image automation | Argo Image Updater | Flux Image Automation |
| Learning curve | Moderate | Steeper (more modular) |

## Repository Structure

**Environment-per-branch** (simpler, less recommended):
```
main → production
staging → staging
```
Risk: branch drift, complex merges.

**Environment-per-directory** (recommended):
```
gitops-config/
  apps/
    my-api/
      base/               # shared Kubernetes YAML
      overlays/
        staging/          # Kustomize patches for staging
        production/       # Kustomize patches for production
  clusters/
    staging/
    production/
  infrastructure/
    cert-manager/
    ingress-nginx/
    monitoring/
```

## Image Update Automation

GitOps needs a way to update image tags without manual PRs:

**Argo Image Updater**:
```yaml
annotations:
  argocd-image-updater.argoproj.io/image-list: myapp=myregistry/myapp
  argocd-image-updater.argoproj.io/myapp.update-strategy: semver
  argocd-image-updater.argoproj.io/myapp.allow-tags: regexp:^v[0-9]+\.[0-9]+\.[0-9]+$
```

**CI/CD Flow with GitOps**:
```
1. Developer pushes code → GitHub
2. CI builds Docker image → pushes to registry with SHA tag
3. CI opens PR: updates image tag in gitops-config repo
4. PR reviewed/auto-merged
5. ArgoCD/Flux detects change → deploys to cluster
```

## Secrets Management

Never store secrets in Git. Patterns:

**Sealed Secrets** (Bitnami): asymmetric encryption; encrypted `SealedSecret` resource stored in Git; controller decrypts in-cluster.

**External Secrets Operator**: syncs secrets from AWS SSM, GCP Secret Manager, HashiCorp Vault into Kubernetes Secrets.

**SOPS + age**: encrypt secret files before committing; Flux has native SOPS decryption.

## Trade-offs

| Pro | Con |
|-----|-----|
| Full audit trail in Git | Bootstrapping complexity |
| Easy rollback (`git revert`) | Secret management is non-trivial |
| No cluster credentials in CI | Debugging reconciliation failures |
| Self-healing drift correction | Config repo can become a bottleneck |
| PR-based change approval | Not ideal for ephemeral/dev environments |

## When to Use

✅ Kubernetes-based production environments  
✅ Teams requiring audit trails and change approval  
✅ Multi-cluster or multi-environment management  
✅ Regulated industries (SOC2, HIPAA audit requirements)  
❌ Non-Kubernetes deployments (GitOps concepts apply but tooling is different)  
❌ Rapid prototyping / dev environments (overhead not worth it)  
❌ Environments where infra changes are very frequent and automated PRs create noise

---
*Related: [[Canary Deployments]], [[Blue-Green Deployments]], [[Observability Stack]]*
