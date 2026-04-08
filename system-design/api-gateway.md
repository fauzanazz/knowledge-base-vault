---
title: "API Gateway"
category: system-design
summary: "An API Gateway is a facade that hides internal microservice APIs behind a unified interface, providing routing, composition, translation, and cross-cutting concerns like authentication and rate limiting."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-08T18:49:29.762Z
---

# API Gateway

> An API Gateway is a facade that hides internal microservice APIs behind a unified interface, providing routing, composition, translation, and cross-cutting concerns like authentication and rate limiting.

# API Gateway

An API Gateway is a facade that hides internal microservice APIs behind a unified interface, providing routing, composition, translation, and cross-cutting concerns like authentication and rate limiting.

## Core Responsibilities

### Routing
Maps public endpoints to internal ones, allowing internal endpoints to change while keeping public APIs stable. Enables long-term API maintenance without breaking clients.

### Composition
Stitches data from multiple services into higher-level APIs, reducing client complexity and network calls. However, availability decreases as the number of composed services increases.

### Translation
Transforms between IPC mechanisms (e.g., gRPC to REST) and provides different APIs for different clients (desktop vs mobile). [[GraphQL]] enables clients to specify exactly what data to fetch.

### Cross-Cutting Concerns
Handles functionality like [[Caching]] static resources, rate limiting, authentication, and authorization.

## Authentication and Authorization

**Authentication** is best handled at the gateway to support multiple mechanisms without service awareness.

**Authorization** is best left to individual services to avoid coupling the gateway with domain logic.

The gateway passes security tokens to internal services:

**Opaque tokens**: Services call external service for user information
**Transparent tokens**: Token contains user information (e.g., JWT)

JWT (JSON Web Tokens) are popular transparent tokens containing expiration date, user identity, roles, and metadata, signed with certificates trusted by internal services.

**API keys** are commonly used for third-party API access.

## Implementation Options

- Build in-house using reverse proxy (e.g., NGINX)
- Use managed solutions (Azure API Management, Amazon API Gateway)

## Caveats

- Can become development bottleneck due to tight coupling with internal APIs
- Requires changes when internal APIs change
- Must scale to handle request rate for all backend services
- Single point of failure requiring high availability design

API gateways are worthwhile investments for applications with many services and APIs.

---
*Related: [[Microservices Architecture]], [[Load Balancer]], [[Caching]], [[Message Queue]]*
