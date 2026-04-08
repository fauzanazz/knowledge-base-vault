---
title: "AutoGen"
category: machine-learning
summary: "AutoGen is Microsoft's conversational multi-agent framework where agents communicate through message exchanges. It features native code execution sandboxes and teachable agents that learn from conversations."
sources:
  - raw/articles/agent-frameworks-langchain-crewai-autogen-swarm.md
updated: 2026-04-08T19:16:25.030Z
---

# AutoGen

> AutoGen is Microsoft's conversational multi-agent framework where agents communicate through message exchanges. It features native code execution sandboxes and teachable agents that learn from conversations.

# AutoGen

Microsoft's AutoGen frames multi-agent systems as **conversations between agents**. Agents send messages to each other, and the conversation history serves as the shared state. It's particularly strong for code-heavy workflows requiring autonomous execution.

## Core Architecture

AutoGen uses two main agent types:

**AssistantAgent**: LLM-powered agents that generate responses and code

**UserProxyAgent**: Executes code locally and returns results to the conversation

```python
from autogen import AssistantAgent, UserProxyAgent

assistant = AssistantAgent(
    name="assistant",
    llm_config={"model": "gpt-4o"},
    system_message="You are a helpful AI assistant."
)

user_proxy = UserProxyAgent(
    name="user_proxy",
    human_input_mode="NEVER",
    code_execution_config={"work_dir": "coding", "use_docker": True}
)

user_proxy.initiate_chat(
    assistant,
    message="Write Python script to analyze this CSV: data.csv"
)
```

## Multi-Agent Conversations

**Group Chat** enables multiple agents to participate in conversations with automatic speaker selection:

```python
from autogen import GroupChat, GroupChatManager

groupchat = GroupChat(
    agents=[user_proxy, planner, coder, reviewer],
    max_round=20,
    speaker_selection_method="auto"  # LLM selects next speaker
)
manager = GroupChatManager(groupchat=groupchat)
```

## Key Features

### Code Execution Sandbox
Docker-isolated Python execution is natively supported, making it ideal for data science and code generation workflows.

### Teachable Agents
Agents can learn from conversations using vector-store backed memory, improving performance over time.

### Nested Chats
Agents can spin up sub-conversations internally for complex reasoning tasks.

## When to Use

AutoGen excels at:
- Data science workflows
- Code generation and self-correction loops
- Any task requiring autonomous code execution
- Scenarios where conversation history is the natural state model

AutoGen Studio provides a no-code UI for building and testing multi-agent conversations.

---
*Related: [[Agent Frameworks]], [[LangGraph]], [[CrewAI]], [[AI Memory Systems]]*
