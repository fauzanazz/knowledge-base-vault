---
title: "Model Context Protocol"
category: machine-learning
summary: "The Model Context Protocol (MCP) is an open standard developed by Anthropic that defines how LLMs communicate with external tools, data sources, and services. It provides a universal interface that works across different models and tools, similar to USB-C for AI."
sources:
  - raw/articles/model-context-protocol-mcp-anthropic.md
updated: 2026-04-08T19:23:19.722Z
---

# Model Context Protocol

> The Model Context Protocol (MCP) is an open standard developed by Anthropic that defines how LLMs communicate with external tools, data sources, and services. It provides a universal interface that works across different models and tools, similar to USB-C for AI.

# Model Context Protocol

The Model Context Protocol (MCP) is an open standard developed by Anthropic and released in November 2024 that defines how LLMs communicate with external tools, data sources, and services. Think of it as USB-C for AI — a single, well-defined connector that works regardless of which model or tool is on either end.

## Architecture

MCP has three distinct roles:

**MCP Host**: The application that contains an LLM and wants to connect it to external capabilities. Claude Desktop is the canonical host, but Cursor IDE, Continue, and custom applications are also hosts.

**MCP Client**: The protocol implementation embedded in the host that manages connections to MCP servers, handles capability negotiation, and translates between the host's internal representation and the MCP wire format.

**MCP Server**: A lightweight process that exposes tools, resources, or prompts over the MCP protocol. Servers can be local processes (communicating over stdio) or remote services (communicating over HTTP/SSE).

## Three Primitives

MCP servers expose three types of capabilities:

**Tools**: Functions the LLM can invoke with name, description, and JSON Schema input specification. Similar to [[Tool Calling]] but standardized across providers.

**Resources**: Data the LLM can read — files, database records, API responses, live system state. Resources are identified by URI and can be read without triggering side effects.

**Prompts**: Server-defined prompt templates with parameter slots that generate pre-structured messages for LLM consumption.

## Transport Layers

MCP defines three transport mechanisms:

- **stdio**: For local servers running as child processes, communicating over stdin/stdout with JSON-RPC messages
- **Server-Sent Events (SSE)**: For remote servers that need to push updates during long-running operations
- **HTTP**: The newest transport supporting stateless request/response and optional streaming for production deployments

## Security Model

Local stdio servers inherit trust from OS process permissions. Remote servers mandate OAuth 2.1 authentication with PKCE flow for secure access. The protocol includes a sampling capability where servers can request LLM calls from hosts, but this should only be enabled for trusted servers.

## Ecosystem

Reference servers exist for common integrations including filesystem access, GitHub API, Slack, PostgreSQL, SQLite, browser automation, and web search. The protocol is MIT-licensed with SDKs available in TypeScript and Python.

MCP operates at a different layer than [[OpenAI Function Calling]] or LangChain tools — it's a transport protocol that those frameworks can build upon for cross-provider compatibility.

---
*Related: [[Tool Calling]], [[Agent Frameworks]], [[API Gateway]], [[LangGraph]]*
