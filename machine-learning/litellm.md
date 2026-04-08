---
title: "LiteLLM"
category: machine-learning
summary: "LiteLLM is an open-source Python library and proxy server that provides an OpenAI-compatible API for 100+ models across providers. It offers routing, load balancing, fallbacks, and caching for production LLM deployments."
sources:
  - raw/articles/ai-gateway-router-litellm-portkey.md
updated: 2026-04-08T19:17:15.395Z
---

# LiteLLM

> LiteLLM is an open-source Python library and proxy server that provides an OpenAI-compatible API for 100+ models across providers. It offers routing, load balancing, fallbacks, and caching for production LLM deployments.

# LiteLLM

LiteLLM is an open-source Python library and proxy server that exposes an OpenAI-compatible REST API in front of 100+ models across providers including OpenAI, Anthropic, Google Gemini, Cohere, Mistral, Azure OpenAI, AWS Bedrock, and self-hosted models via Ollama or vLLM.

## Library Mode

In library mode, LiteLLM allows swapping OpenAI SDK calls without changing the call signature:

```python
import litellm

response = litellm.completion(
    model="anthropic/claude-3-5-sonnet-20241022",
    messages=[{"role": "user", "content": "Explain backpressure"}],
    max_tokens=512,
)
```

LiteLLM normalizes response formats across providers, always returning OpenAI-shaped objects.

## Proxy Server Mode

The proxy exposes a drop-in OpenAI-compatible endpoint:

```bash
pip install litellm[proxy]
litellm --model anthropic/claude-3-5-sonnet-20241022 --port 8000
```

Applications point at `http://localhost:8000` and send standard OpenAI requests.

## Router: Core Orchestration

LiteLLM's Router handles multi-provider deployments with [[Load Balancer|load balancing]] and fallbacks:

```python
from litellm import Router

router = Router(
    model_list=[
        {
            "model_name": "gpt-4o",
            "litellm_params": {
                "model": "openai/gpt-4o",
                "rpm": 500,
                "tpm": 150000
            },
        },
        {
            "model_name": "gpt-4o",
            "litellm_params": {
                "model": "azure/gpt-4o",
                "rpm": 1000,
            },
        },
    ],
    routing_strategy="least-busy",
    fallbacks=[{"gpt-4o": ["claude-fallback"]}],
)
```

## Routing Strategies

- **Cost-based routing**: Routes to cheapest models first
- **Latency-based routing**: Tracks P50 latency and routes to fastest deployments
- **Least-busy**: Routes to deployments with lowest current load
- **Weighted routing**: Distributes traffic by configured percentages

## Enterprise Features

- **[[Caching]]**: Redis-backed exact-match and semantic caching
- **Budget management**: Per-team spending limits and quotas
- **Audit logging**: Integration with Langfuse, Prometheus, DataDog
- **Rate limiting**: TPM/RPM limits per deployment

LiteLLM serves as the foundation for many production [[AI Gateway]] deployments, offering developer-friendly APIs with enterprise-grade reliability features.

---
*Related: [[AI Gateway]], [[Portkey]], [[Load Balancer]], [[Caching]], [[Vector Embeddings]]*
