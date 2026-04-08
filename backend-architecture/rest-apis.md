---
title: "REST APIs"
category: api-design
summary: "REST APIs are HTTP-based interfaces that follow design principles including statelessness and cacheability, using URLs to identify resources and HTTP methods to define operations."
sources:
  - raw/articles/communication-understanding-distributed-systems-by-roberto-vitillo-pagefy.md
updated: 2026-04-04T10:17:50.260Z
---

# REST APIs

> REST APIs are HTTP-based interfaces that follow design principles including statelessness and cacheability, using URLs to identify resources and HTTP methods to define operations.

# REST APIs

REST (Representational State Transfer) APIs are HTTP-based interfaces that follow specific design principles for creating elegant and scalable web services.

## Core Principles

- **Stateless requests**: Each request contains all necessary information for processing
- **Cacheable responses**: Responses are explicitly or implicitly labeled as cacheable
- **Resource-based**: URLs identify resources rather than actions

## Resource Design

REST APIs model resources as entities that support CRUD operations:
- `POST /products` - Create new product
- `GET /products` - List all products (with query parameters for filtering)
- `GET /products/42` - Retrieve specific product
- `PUT /products/42` - Update specific product
- `DELETE /products/42` - Delete specific product

URLs can model resource relationships: `/products/42/reviews` represents reviews for product 42.

## Data Serialization

Common formats include:
- **JSON**: Human-readable but larger payloads and parsing overhead
- **Protocol Buffers**: Binary format with smaller payloads but not human-readable

Content type is specified via `Content-Type` header, typically `application/json`.

## API Evolution

APIs must evolve without breaking existing clients:
- **Versioning**: Use URL versioning (e.g., `/v1/products`)
- **Backward compatibility**: Prefer additive changes over breaking changes
- **Schema evolution**: Avoid changing property types or making optional parameters required

## Idempotency

Idempotent operations can be safely retried. Non-idempotent operations like POST can be made idempotent using:
- **Idempotency keys**: Client-generated UUIDs in headers
- **Server-side deduplication**: Store keys in database with TTL
- **Atomic operations**: Ensure key storage and request execution are atomic

REST APIs provide the foundation for scalable web services in modern [[Distributed Systems]] architectures.

---
*Related: [[HTTP Protocol]], [[API Design]], [[Idempotency]], [[Web Services]], [[Distributed Systems]]*
