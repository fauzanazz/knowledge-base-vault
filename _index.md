# system-design

> Knowledge base with 89 articles across 13 categories
> Last compiled: 2026-04-08T19:32:50.527Z

## algorithms

- [[HNSW Algorithm]] — HNSW (Hierarchical Navigable Small World) is the dominant algorithm for approximate nearest-neighbor search in vector databases. It builds multi-layer graphs enabling fast traversal with 95-99% recall and ~10ms query times on millions of vectors.

## architecture

- [[Software Architecture Principles]] — Software architecture principles guide system design decisions through established patterns like twelve-factor apps, reactive systems, and microservices. Good architecture balances scalability, maintainability, and complexity.

## database

- [[ACID Properties]] — ACID properties define the fundamental guarantees that database transactions must provide: Atomicity, Consistency, Isolation, and Durability.
- [[Chroma]] — Chroma is an open-source, local-first vector database designed for easy prototyping and development. It features zero-config setup, built-in embedding functions, and can run in-memory, locally persistent, or distributed modes.
- [[Database Replication]] — Database replication creates copies of data across multiple database servers to improve performance, reliability, and availability. The most common pattern is the master-slave model where writes go to the master and reads are distributed across slaves.
- [[Database Sharding]] — Database sharding is a horizontal scaling technique that divides large databases into smaller, more manageable parts called shards. Each shard contains a subset of data and shares the same schema but stores unique data.
- [[Dynamo-Style Data Stores]] — Dynamo-style data stores are eventually consistent, highly available key-value stores that use quorum-based replication and conflict resolution through CRDTs.
- [[Hash Partitioning]] — Hash partitioning uses hash functions to deterministically map keys to partitions, providing uniform distribution but losing sorted order. Consistent hashing enables efficient rebalancing.
- [[Isolation Levels]] — Isolation levels define the degree to which transactions are isolated from each other, balancing consistency guarantees against performance through different concurrency control mechanisms.
- [[NewSQL Databases]] — NewSQL databases combine the ACID guarantees of traditional SQL databases with the horizontal scalability of NoSQL systems through advanced distributed consensus and partitioning techniques.
- [[NoSQL Databases]] — NoSQL databases are designed for horizontal scalability and high availability, trading ACID guarantees for partition tolerance. They store data as key-value pairs or documents without strict schemas.
- [[Pinecone]] — Pinecone is a fully managed vector database service that provides serverless and pod-based deployments for storing and querying high-dimensional embeddings. It offers zero ops overhead with auto-scaling capabilities.
- [[Qdrant]] — Qdrant is an open-source vector database written in Rust, designed for high performance and reliability. It offers rich filtering capabilities, named vectors, and sparse vector support for hybrid search applications.
- [[Range Partitioning]] — Range partitioning splits data into lexicographically sorted partitions, enabling efficient range queries but facing challenges with uneven data distribution and hotspots.
- [[Vector Databases]] — Vector databases are specialized storage systems optimized for storing, indexing, and querying high-dimensional vector embeddings. They enable fast similarity search and are essential for AI applications like semantic search and RAG systems.

## distributed-systems

- [[Broadcast Protocols]] — Broadcast protocols enable message delivery to multiple receivers in distributed systems, with different guarantees for reliability and ordering.
- [[CALM Theorem]] — The CALM theorem states that monotonic programs can achieve consistency without coordination, providing a framework for building coordination-free distributed applications.
- [[Causal Consistency]] — Causal consistency preserves happens-before relationships between operations while allowing concurrent operations to be observed in different orders, providing a middle ground between eventual and strong consistency.
- [[Chain Replication]] — Chain replication arranges processes in a linear chain where writes flow from head to tail, providing strong consistency with different trade-offs than leader-based replication.
- [[Conflict-Free Replicated Data Types]] — CRDTs are data structures that automatically resolve conflicts in distributed systems without requiring consensus, enabling strong eventual consistency through mathematical properties.
- [[Consistency Models]] — Consistency models define trade-offs between data consistency and system performance in distributed systems, ranging from strong to eventual consistency.
- [[Failure Detection]] — Failure detection mechanisms help distributed systems identify when processes have crashed or become unavailable through timeouts, pings, and heartbeats.
- [[Leader Election]] — Leader election algorithms select a single process from a group to coordinate shared resources or assign work, ensuring safety and liveness properties.
- [[Logical Clocks]] — Logical clocks measure the passing of time in terms of logical operations rather than wall-clock time, enabling event ordering in distributed systems.
- [[Outbox Pattern]] — The Outbox pattern ensures consistency between multiple data stores by using a local outbox table and asynchronous message processing to guarantee reliable data synchronization.
- [[Physical Clocks]] — Physical clocks provide wall-time timestamps in distributed systems but face synchronization challenges due to clock drift and network latency.
- [[Raft Algorithm]] — Raft is a consensus algorithm that provides strong consistency through leader-based replication, designed to be simpler to understand than Paxos.
- [[Saga Pattern]] — The Saga pattern implements distributed transactions as a sequence of local transactions with compensating actions, providing atomicity without the blocking behavior of two-phase commit.
- [[System Models]] — System models define assumptions about process behavior, communication links, and timing in distributed systems to enable reasoning about correctness and failure scenarios.
- [[Two-Phase Commit]] — Two-phase commit is a distributed consensus protocol that ensures atomic transactions across multiple processes through a coordinator-participant model with prepare and commit phases.

## due-diligence

- [[Technical Due Diligence]] — Technical due diligence evaluates a company's technology assets, architecture, team, and processes during investment or acquisition scenarios. It assesses technical risks, scalability potential, and development capabilities.

## hiring

- [[Engineering Hiring Process]] — Engineering hiring involves systematic evaluation of technical skills, problem-solving ability, and cultural fit through structured interviews, coding assessments, and behavioral evaluations. Effective processes balance thoroughness with candidate experience.

## leadership

- [[Chief Technology Officer Role]] — The Chief Technology Officer (CTO) role varies significantly across organizations, from hands-on technical leadership in startups to strategic technology vision in large enterprises. CTOs must balance technical expertise with business acumen and people management skills.

## machine-learning

- [[Agent Communication Patterns]] — Agent communication patterns define how multiple AI agents exchange information and coordinate work through shared state, message passing, events, and direct calls.
- [[Agent Frameworks]] — Agent frameworks provide infrastructure for building LLM-powered agents that can reason, act, and iterate through tool usage. Major frameworks include LangChain/LangGraph, CrewAI, AutoGen, and OpenAI Swarm.
- [[AI Gateway]] — AI gateways provide a unified interface between applications and multiple AI providers, solving vendor lock-in, cost optimization, reliability, and observability challenges. They act as load balancers for LLMs without changing application API contracts.
- [[AI Guardrails]] — AI guardrails are safety mechanisms that filter and validate inputs and outputs in production LLM systems to prevent hallucination, PII leakage, prompt injection, and other failure modes through multi-layered defense strategies.
- [[AI Memory Systems]] — AI memory systems externalize state from stateless LLMs into queryable storage, enabling persistent learning and breaking context window limitations. They mirror human memory taxonomy with short-term, episodic, semantic, and procedural memory types.
- [[AutoGen]] — AutoGen is Microsoft's conversational multi-agent framework where agents communicate through message exchanges. It features native code execution sandboxes and teachable agents that learn from conversations.
- [[Braintrust]] — Braintrust is an experiment tracking platform for LLMs that focuses on comparing metric distributions across prompt versions and model changes, with built-in caching proxy for cost optimization.
- [[Chain-of-Thought]] — Chain-of-Thought (CoT) prompting asks LLMs to show their reasoning process before producing final answers, dramatically improving accuracy on multi-step reasoning tasks. It can be implemented as zero-shot or few-shot approaches.
- [[Chunking Strategies]] — Chunking strategies determine retrieval granularity in RAG systems and are the most common failure point. Strategies range from fixed-size splitting to semantic chunking and parent-child hierarchies.
- [[CrewAI]] — CrewAI is a role-based multi-agent framework that models agent systems as crews of specialized agents with defined roles, goals, and backstories. Agents can delegate tasks to each other in sequential or hierarchical processes.
- [[Cross-Encoder Reranking]] — Cross-encoder reranking is a post-retrieval optimization that jointly scores query-document pairs for improved precision. It's more accurate than bi-encoder similarity but has O(N) inference cost.
- [[DeepEval]] — DeepEval is a general-purpose LLM evaluation framework with 14+ metrics and pytest integration, covering quality dimensions beyond RAG including bias, toxicity, and custom criteria evaluation.
- [[Few-Shot Learning]] — Few-shot learning uses a small number of examples (typically 3-5) to teach LLMs patterns for structured tasks, providing higher ROI than exhaustive instructions. It's the most effective prompt enhancement for classification and structured generation.
- [[Fine-tuning vs RAG vs Prompt Engineering]] — A decision framework for choosing between prompt engineering, RAG, and fine-tuning when customizing LLM behavior based on cost, effort, and performance trade-offs.
- [[Guardrails AI Framework]] — Guardrails AI is a Python framework that provides structured validation for LLM inputs and outputs through a hub of 50+ pre-built validators and automatic retry-with-correction capabilities.
- [[HyDE]] — HyDE (Hypothetical Document Embedding) is a retrieval technique that generates fake answers to queries and uses their embeddings for search. It works well when query language differs from document language.
- [[LangGraph]] — LangGraph is a production-grade agent framework that models workflows as directed graphs with nodes as functions and edges as transitions. It supports state persistence, human-in-the-loop checkpoints, and complex branching logic.
- [[LiteLLM]] — LiteLLM is an open-source Python library and proxy server that provides an OpenAI-compatible API for 100+ models across providers. It offers routing, load balancing, fallbacks, and caching for production LLM deployments.
- [[LlamaGuard]] — LlamaGuard is Meta's fine-tuned Llama model specialized for content safety classification, covering 14 hazard categories and enabling self-hosted safety validation without third-party data exposure.
- [[LLM Evaluation Frameworks]] — LLM evaluation frameworks provide systematic, reproducible metrics to replace vibes-based testing with quantified quality assurance for language model applications.
- [[LLM Fine-tuning]] — Fine-tuning continues training pre-trained language models on domain-specific data to permanently alter weights and internalize specialized patterns, style, and behaviors.
- [[Mem0]] — Mem0 is a managed memory layer that provides automatic extraction and retrieval of memories from LLM conversations. It sits transparently between applications and LLMs, using LLMs to extract atomic memory facts with deduplication and relevance scoring.
- [[Model Context Protocol]] — The Model Context Protocol (MCP) is an open standard developed by Anthropic that defines how LLMs communicate with external tools, data sources, and services. It provides a universal interface that works across different models and tools, similar to USB-C for AI.
- [[Multi-Agent Orchestration Patterns]] — Multi-agent orchestration patterns provide structured approaches for coordinating multiple LLM agents to achieve complex tasks through specialization, parallelism, and collaboration.
- [[NeMo Guardrails]] — NeMo Guardrails is NVIDIA's dialog-centric safety framework that uses Colang domain-specific language to define conversation flow constraints and multi-turn safety logic for LLM applications.
- [[OpenAI Swarm]] — OpenAI Swarm is a minimalist agent framework built on two primitives: agents (LLMs with instructions and tools) and handoffs (transferring control between agents). It's designed for prototyping rather than production use.
- [[Portkey]] — Portkey is a managed AI gateway that combines routing with observability, virtual keys, and UI-driven configuration. It provides enterprise-oriented features for managing LLM deployments at scale.
- [[Program-of-Thought]] — Program-of-Thought (PoT) has LLMs write and execute code instead of reasoning in natural language, providing more reliable results for numerical and logical tasks through deterministic execution.
- [[Prompt Engineering]] — Prompt engineering is the discipline of crafting LLM inputs to reliably produce high-quality outputs through systematic design, versioning, testing, and cost control. It focuses on production-grade patterns rather than clever tricks.
- [[Prompt Injection Detection]] — Prompt injection detection identifies adversarial inputs that attempt to override system instructions in LLM applications, using classifier-based, heuristic, or LLM-judge approaches to prevent security vulnerabilities.
- [[RAG Architecture]] — Retrieval-Augmented Generation (RAG) grounds LLM responses in external knowledge through a spectrum of architectures from simple retrieve-then-read pipelines to modular orchestration systems. It reduces hallucination and enables domain-specific answers without fine-tuning.
- [[RAGAS]] — RAGAS is an evaluation framework for RAG systems that measures faithfulness, answer relevance, context precision, and context recall. It provides standardized metrics for assessing retrieval and generation quality.
- [[ReAct Pattern]] — ReAct (Reasoning + Acting) is the foundational pattern for LLM agents where models alternate between thinking, acting through tool calls, and observing results until reaching a goal. Most agent frameworks implement variants of this pattern.
- [[Tool Calling]] — Tool calling (also called function calling) is the mechanism by which LLMs emit structured JSON requests for external actions rather than free-form text. It's the foundational primitive for building agents and automation pipelines.
- [[Tree of Thought]] — Tree of Thought (ToT) extends Chain-of-Thought by exploring multiple reasoning branches in parallel, evaluating each path, and selecting the best solution. It's compute-intensive but valuable for complex planning and problem-solving tasks.
- [[Vector Embeddings]] — Vector embeddings are dense, fixed-length numerical representations that encode semantic meaning of inputs like text, images, or audio. Semantically similar inputs produce geometrically close vectors in high-dimensional space.
- [[Zep]] — Zep is an open-source long-term memory server optimized for chat applications. It provides automatic message summarization, named entity extraction, temporal knowledge graphs, and session-level memory scopes through REST APIs.

## management

- [[Engineering Management]] — Engineering management involves leading technical teams through a combination of people leadership, technical guidance, and process optimization. Successful engineering managers balance individual contributor skills with management responsibilities.

## process

- [[Development Process]] — Development processes provide structured approaches to software delivery through methodologies like Agile/Scrum, continuous integration/deployment, and version control workflows. Effective processes balance structure with team autonomy.

## search

- [[Hybrid Search]] — Hybrid search combines dense vector embeddings with sparse keyword-based vectors to leverage both semantic understanding and exact keyword matching. It consistently outperforms dense-only search through techniques like Reciprocal Rank Fusion.

## startups

- [[Startup Technology Strategy]] — Startup technology strategy focuses on rapid iteration, technical debt management, and scaling decisions that balance speed-to-market with long-term sustainability. CTOs must navigate resource constraints while building foundations for growth.

## system-design

- [[API Gateway]] — An API Gateway is a facade that hides internal microservice APIs behind a unified interface, providing routing, composition, translation, and cross-cutting concerns like authentication and rate limiting.
- [[Blob Storage Architecture]] — Blob storage systems like Azure Storage use multi-layer architectures with stream, partition, and front-end layers to provide scalable, durable file storage across geographically distributed clusters.
- [[Caching]] — Caching is a technique that stores frequently accessed data in fast memory storage to reduce database load and improve response times. It serves as a temporary data store layer that is much faster than traditional databases.
- [[Content Delivery Network]] — A Content Delivery Network (CDN) is a geographically distributed network of servers that cache static content closer to users to improve load times and reduce server load. CDNs are essential for global applications requiring fast content delivery.
- [[Control Plane and Data Plane]] — Control plane and data plane separation is a common pattern where the data plane handles critical path requests requiring high availability and scale, while the control plane manages configuration and metadata with different requirements.
- [[Fanout Service]] — A fanout service propagates user posts to their friends' news feeds, implementing either push, pull, or hybrid strategies based on user connection patterns. It's a critical component for distributing content at scale in social media systems.
- [[Horizontal Scaling]] — Horizontal scaling involves adding more servers to a system pool rather than upgrading existing hardware. This approach is more suitable for large-scale systems and requires load balancers to distribute traffic across multiple servers.
- [[HTTP Caching]] — HTTP caching enables browsers and servers to store static resources locally to reduce network calls and improve performance. It uses Cache-Control headers and ETags to manage resource freshness and validation.
- [[Load Balancer]] — A load balancer distributes incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. It provides redundancy, scalability, and improved system reliability.
- [[Message Queue]] — A message queue is a durable component that enables asynchronous communication between system components. It serves as a buffer that stores messages in memory and allows producers to send messages that consumers can process independently.
- [[Microservices Architecture]] — Microservices architecture decomposes monolithic applications into independently deployable services communicating via APIs. This approach enables better team autonomy and scalability but introduces distributed system complexity.
- [[Multi-Data Center Setup]] — Multi-data center setup involves deploying applications across multiple geographically distributed data centers to improve availability, reduce latency, and provide disaster recovery capabilities. It requires careful consideration of traffic routing, data synchronization, and deployment strategies.
- [[News Feed System]] — A news feed system displays constantly updating posts from user connections in reverse chronological order. It requires careful design of feed publishing and retrieval flows to handle millions of daily active users.
- [[System Scaling]] — System scaling is the process of growing a system architecture from a single server to handle millions of users through iterative refinement and optimization. It involves both vertical and horizontal scaling strategies to manage increasing traffic and data loads.
- [[Vertical Scaling]] — Vertical scaling involves adding more resources (CPU, RAM, storage) to existing servers to handle increased load. While simpler to implement than horizontal scaling, it has limitations including hardware constraints, lack of redundancy, and higher costs.

