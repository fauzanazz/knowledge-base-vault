---
title: "API Gateway"
category: system-design
summary: "An API Gateway is a facade that hides internal microservice APIs behind a unified interface, providing routing, composition, translation, and cross-cutting concerns like authentication and rate limiting."
sources:
  - raw/articles/scalability-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:12:09.754Z
---

# API Gateway

> An API Gateway is a facade that hides internal microservice APIs behind a unified interface, providing routing, composition, translation, and cross-cutting concerns like authentication and rate limiting.

# API Gateway

An API Gateway is a facade that hides internal [[Microservices Architecture]] APIs behind a unified public interface. It solves problems like expensive mobile operations requiring multiple API calls and tight coupling between clients and internal service details.

## Core Responsibilities

### Routing
Maps public endpoints to internal ones, allowing internal endpoints to change while keeping public APIs stable. This is crucial since public APIs must be maintained for extended periods.

### Composition
Stitches data from multiple internal services into higher-level APIs, reducing client complexity and the number of required calls. However, availability decreases as the number of composed services increases.

### Translation
Transforms between different IPC mechanisms (e.g., gRPC to REST) and exposes different APIs for different clients (desktop vs mobile). [[GraphQL]] is a popular graph-based API that lets clients decide how much data to fetch.

### Cross-Cutting Concerns
Handles functionality like [[Caching]] static resources, rate limiting, and most critically, authentication and authorization (authN/authZ).

## Authentication and Authorization

**Authentication** is best handled at the API Gateway to support multiple authentication mechanisms without service awareness.

**Authorization** is best left to individual services to avoid coupling the gateway with domain logic.

The gateway passes security tokens to internal services:

- **Opaque Tokens**: Services call external services for user information
- **Transparent Tokens**: Contain user information directly (e.g., [[JWT]])

Transparent tokens avoid external calls but are harder to revoke if compromised.

**API Keys** are popular for third-party API access.

## Implementation Challenges

- **Development Bottleneck**: Tightly coupled with internal APIs it proxies
- **Change Propagation**: Internal API changes require gateway updates
- **Scaling Requirements**: Must scale to handle request rates for all backend services
- **Single Point of Failure**: Requires high availability and scalability

## Implementation Options

- Build in-house using reverse proxies like NGINX
- Use managed solutions like Azure API Management or Amazon API Gateway

API Gateways are worthwhile investments for applications with many services and APIs.

---
*Related: [[Microservices Architecture]], [[JWT]], [[GraphQL]], [[Caching]], [[Load Balancer]]*
