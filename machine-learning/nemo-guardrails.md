---
title: "NeMo Guardrails"
category: machine-learning
summary: "NeMo Guardrails is NVIDIA's dialog-centric safety framework that uses Colang domain-specific language to define conversation flow constraints and multi-turn safety logic for LLM applications."
sources:
  - raw/articles/guardrails-safety-input-output-filtering.md
updated: 2026-04-08T19:22:50.919Z
---

# NeMo Guardrails

> NeMo Guardrails is NVIDIA's dialog-centric safety framework that uses Colang domain-specific language to define conversation flow constraints and multi-turn safety logic for LLM applications.

# NeMo Guardrails

NeMo Guardrails is NVIDIA's dialog-centric safety framework that uses Colang, a domain-specific language, to define conversation flow constraints and multi-turn safety logic for LLM applications.

## Colang Rails Definition

NeMo uses Colang to express safety rules as dialog flows:

```colang
# Define prohibited topics
define user ask politics
  "What's your opinion on [political party]?"
  "Who should I vote for?"

define bot refuse politics
  "I'm not able to discuss political topics."

# Wire them together
define flow politics guardrail
  user ask politics
  bot refuse politics
```

## Multi-Turn Safety Logic

Unlike simple input/output filters, NeMo excels at complex conversational constraints:
- Escalation rules ("if user asks about X more than 3 times, transfer to human")
- Context-aware responses based on conversation history
- Topic drift detection across multiple exchanges
- User behavior pattern recognition

## Topical Rails

Topical rails keep conversations within defined boundaries:

```colang
define flow off topic
  user ...
  $allowed = execute check_allowed_topics(topic=$last_user_message)
  if not $allowed
    bot refuse to respond
```

## Integration Capabilities

NeMo integrates with popular frameworks:
- LangChain pipelines for [[Agent Frameworks]] workflows
- LlamaIndex for [[RAG Architecture]] applications
- Custom LLM providers through standardized interfaces

## Production Considerations

NeMo adds 100-400ms latency through its in-process Colang engine. The dialog-centric approach makes it ideal for chatbots, customer service applications, and any system requiring sophisticated conversation management.

## Use Cases

NeMo Guardrails is particularly valuable for:
- Customer service bots with escalation workflows
- Educational applications with topic boundaries
- Healthcare chatbots requiring strict safety protocols
- Enterprise assistants with role-based access controls

It complements other [[AI Guardrails]] frameworks by focusing specifically on dialog flow control rather than content filtering.

---
*Related: [[AI Guardrails]], [[Agent Frameworks]], [[Prompt Engineering]]*
