---
title: "Zep"
category: machine-learning
summary: "Zep is an open-source long-term memory server optimized for chat applications. It provides automatic message summarization, named entity extraction, temporal knowledge graphs, and session-level memory scopes through REST APIs."
sources:
  - raw/articles/ai-memory-systems-short-long-term-episodic-semantic.md
updated: 2026-04-08T19:15:07.373Z
---

# Zep

> Zep is an open-source long-term memory server optimized for chat applications. It provides automatic message summarization, named entity extraction, temporal knowledge graphs, and session-level memory scopes through REST APIs.

# Zep

Zep is an open-source long-term memory server designed specifically for chat applications and conversational AI. It provides comprehensive memory management capabilities including automatic summarization, entity extraction, and knowledge graph construction.

## Core Features

**Automatic Message Summarization**: Zep automatically compresses conversation history into concise summaries while preserving important context and details.

**Named Entity Extraction**: The system identifies and tracks entities (people, places, organizations) mentioned in conversations, building a structured understanding of context.

**Temporal Knowledge Graphs**: Zep constructs knowledge graphs that capture relationships between entities and events over time, enabling complex reasoning about conversation history.

**Memory Scopes**: Supports both session-level memories (specific to individual conversations) and user-level memories (persistent across all sessions for a user).

## API Integration

```python
from zep_cloud.client import Zep

zep = Zep(api_key="...")
zep.memory.add(session_id, messages=messages)
memory = zep.memory.get(session_id)  # Returns summary + entities + relevant facts
```

## Architecture

Zep operates as a standalone memory server with REST APIs and SDKs for Python and TypeScript. This architecture allows multiple applications to share memory state and enables horizontal scaling of memory operations.

## Use Cases

Zep excels in applications requiring rich conversational context like customer support bots, personal assistants, and multi-session workflows. Its knowledge graph capabilities make it particularly suitable for applications needing to track complex relationships and temporal sequences.

As part of broader [[AI Memory Systems]], Zep handles the episodic memory layer while integrating with [[Vector Databases]] for semantic memory and [[RAG Architecture]] for knowledge retrieval.

---
*Related: [[AI Memory Systems]], [[Mem0]], [[Vector Databases]], [[RAG Architecture]]*
