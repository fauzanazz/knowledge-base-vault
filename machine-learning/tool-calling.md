---
title: "Tool Calling"
category: machine-learning
summary: "Tool calling (also called function calling) is the mechanism by which LLMs emit structured JSON requests for external actions rather than free-form text. It's the foundational primitive for building agents and automation pipelines."
sources:
  - raw/articles/tool-calling-function-calling-llm-api-bridge.md
updated: 2026-04-08T19:18:54.556Z
---

# Tool Calling

> Tool calling (also called function calling) is the mechanism by which LLMs emit structured JSON requests for external actions rather than free-form text. It's the foundational primitive for building agents and automation pipelines.

# Tool Calling

Tool calling (also called function calling) is the mechanism by which LLMs emit structured requests for external actions rather than free-form text. Instead of saying "you should check the weather," a model emits `{"name": "get_weather", "arguments": {"city": "London"}}` — a JSON object your application can deterministically route and execute.

## Core Mechanics

**OpenAI Function Calling** accepts a `tools` array with JSON Schema definitions. The model response contains `tool_calls` with name and arguments when `finish_reason == "tool_calls"`.

**Anthropic Tool Use** uses similar `tools` parameter but returns tool calls as content blocks with `stop_reason == "tool_use"`. Tool results are injected as `user` messages with `tool_result` type.

## Parallel Tool Calls

Both OpenAI (gpt-4o+) and Anthropic (Claude 3+) support returning multiple tool calls in a single response. Always execute concurrent tool calls with `asyncio.gather` for efficiency — don't execute sequentially.

## Tool Selection Strategies

- `tool_choice: "auto"` — Model decides whether to call tools (default)
- `tool_choice: "required"` — Model must call at least one tool
- Force specific function — Useful for structured output extraction

## Security Considerations

Tool results are untrusted input that can contain prompt injection attacks. Mitigations include:
- Sanitizing tool results and wrapping in XML tags
- Sandboxing code execution tools with containers
- Input validation against schemas before execution
- Never following instructions found within tool results

## Production Patterns

Tool definitions consume input tokens on every request. For large tool sets, use dynamic tool selection to show only relevant tools. Cache deterministic, idempotent tool calls with appropriate TTLs. Monitor per-tool error rates and latency to detect anomalies.

Tool calling enables [[Agent Frameworks]] to interact with external systems and is essential for [[RAG Architecture]] systems that need to query knowledge bases.

---
*Related: [[Agent Frameworks]], [[RAG Architecture]], [[ReAct Pattern]], [[AI Gateway]]*
