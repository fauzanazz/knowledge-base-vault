---
title: "Portkey"
category: machine-learning
summary: "Portkey is a managed AI gateway that combines routing with observability, virtual keys, and UI-driven configuration. It provides enterprise-oriented features for managing LLM deployments at scale."
sources:
  - raw/articles/ai-gateway-router-litellm-portkey.md
updated: 2026-04-08T19:17:15.415Z
---

# Portkey

> Portkey is a managed AI gateway that combines routing with observability, virtual keys, and UI-driven configuration. It provides enterprise-oriented features for managing LLM deployments at scale.

# Portkey

Portkey is a managed [[AI Gateway]] that combines routing with first-class observability, virtual keys, and a UI-driven configuration model. It's designed as an enterprise-oriented counterpart to [[LiteLLM]]'s developer-first approach.

## Virtual Keys

Portkey introduces **virtual keys** — an indirection layer between applications and actual provider credentials. Applications hold Portkey virtual keys while Portkey securely stores real API keys.

```python
import portkey_ai

client = portkey_ai.Portkey(
    api_key="PORTKEY_API_KEY",
    virtual_key="OPENAI_VIRTUAL_KEY",  # maps to real OpenAI key
)

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Hello"}]
)
```

**Benefits of Virtual Keys:**
- Key rotation happens in Portkey UI, not deployment pipelines
- Scope keys to teams, cost centers, or projects
- Instant revocation capabilities
- Centralized credential management

## Config-Driven Routing

Portkey routing uses declarative JSON configuration that can be swapped without code changes:

```json
{
  "strategy": { "mode": "fallback" },
  "targets": [
    {
      "virtual_key": "openai-prod",
      "override_params": { "model": "gpt-4o" },
      "weight": 0.8
    },
    {
      "virtual_key": "anthropic-prod",
      "override_params": { "model": "claude-3-5-sonnet-20241022" },
      "weight": 0.2,
      "on_status_codes": [429, 500, 502, 503]
    }
  ]
}
```

This configuration handles both [[Load Balancer|load balancing]] (80/20 weight split) and fallback (on 5xx or rate limit codes) in a single declaration.

## Enterprise Features

**Observability Dashboard**: Real-time metrics for cost-per-team, P95 latency by model, error rates by provider, and token throughput.

**Budget Management**: Team-level spending limits with graceful degradation when budgets are exceeded.

**A/B Testing**: Traffic splitting between model versions with automatic tagging for quality correlation.

**Audit Logging**: Complete request/response logging with configurable retention and export capabilities.

## Semantic [[Caching]]

Portkey supports semantic caching that embeds queries and returns cached responses for semantically similar requests:

- Configurable similarity thresholds (typically 0.95)
- Catches paraphrases while avoiding incorrect cache hits
- Significant cost reduction for workloads with query similarity

Portkey excels in enterprise environments requiring UI-driven management, comprehensive observability, and non-technical team access to LLM infrastructure controls.

---
*Related: [[AI Gateway]], [[LiteLLM]], [[Load Balancer]], [[Caching]], [[API Gateway]]*
