---
title: "Backend for Frontend"
category: backend-architecture
summary: "An API design pattern where dedicated backend services are created for each client type (mobile, web, third-party) — tailoring response shapes, aggregation logic, and communication protocols to each consumer's specific needs."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Backend for Frontend

> An API design pattern where dedicated backend services are created for each client type (mobile, web, third-party) — tailoring response shapes, aggregation logic, and communication protocols to each consumer's specific needs.

## Overview

The **Backend for Frontend (BFF)** pattern, coined by Sam Newman (author of *Building Microservices*), addresses the mismatch between a general-purpose API and the specific needs of different clients. Rather than one API that tries to serve mobile apps, web apps, and third-party partners equally, each client type gets a dedicated backend tailored to its requirements.

SoundCloud and Netflix pioneered this pattern. Netflix maintains separate BFF layers for their TV/set-top box apps, mobile apps, and web — each with optimized payload shapes, compression, and protocol choices.

## The Problem: One-Size-Fits-All API

```
❌ Single API serving all clients:

Mobile App (low bandwidth, small screen)
  needs: minimal data, compressed, aggressive caching

Web App (full browser, large screen)
  needs: rich data, real-time updates, full feature set

Smart TV App (limited CPU, large screen)
  needs: pre-formatted content, specific image sizes

Third-party API consumers
  needs: stable versioned contracts, extensive documentation

Single API tries to satisfy all → compromises everywhere
Mobile gets too much data; Web doesn't get enough detail
```

## The BFF Solution

```
✅ BFF Pattern:

Mobile App  ──► [Mobile BFF]  ──────────────┐
Web App     ──► [Web BFF]     ──────────────┤──► Downstream Microservices
Smart TV    ──► [TV BFF]      ──────────────┤    (Order, User, Product, etc.)
Partners    ──► [Partner API] ──────────────┘
```

Each BFF:
- Aggregates data from multiple downstream services
- Shapes responses for its specific client
- Handles client-specific auth flows
- Manages client-specific caching strategies

## BFF Implementation Example

### Mobile BFF (optimized for bandwidth)
```typescript
// Mobile BFF: compact product response
app.get('/api/mobile/products/:id', async (req, res) => {
  const [product, inventory, rating] = await Promise.all([
    productService.get(req.params.id),
    inventoryService.stock(req.params.id),
    reviewService.averageRating(req.params.id)
  ]);

  // Mobile: minimal payload, no large description, compressed images
  res.json({
    id: product.id,
    name: product.name,
    price: product.price,
    thumbnail: product.images.thumbnail,  // 200x200 only
    inStock: inventory.quantity > 0,
    rating: rating.average,
    ratingCount: rating.count
  });
});
```

### Web BFF (optimized for richness)
```typescript
// Web BFF: rich product response
app.get('/api/web/products/:id', async (req, res) => {
  const [product, inventory, reviews, recommendations, seo] = await Promise.all([
    productService.get(req.params.id),
    inventoryService.full(req.params.id),     // includes warehouse, eta
    reviewService.all(req.params.id),         // full reviews list
    recommendationService.related(req.params.id),
    seoService.metadata(req.params.id)
  ]);

  // Web: rich payload with all details
  res.json({
    id: product.id,
    name: product.name,
    description: product.fullDescription,    // full description
    price: product.price,
    images: product.images.all,              // all image sizes
    inventory: { quantity: inventory.quantity, eta: inventory.deliveryEta },
    reviews: reviews,                        // all reviews
    recommendations: recommendations,
    seo: seo
  });
});
```

## GraphQL as a Universal BFF

GraphQL can serve as a BFF by letting each client **declare exactly what data it needs**:

```graphql
# Mobile query — minimal payload
query MobileProductView($id: ID!) {
  product(id: $id) {
    id
    name
    price
    thumbnail: image(size: SMALL)
    inStock
  }
}

# Web query — full payload
query WebProductView($id: ID!) {
  product(id: $id) {
    id
    name
    fullDescription
    price
    images { small medium large }
    inventory { quantity deliveryEta warehouse }
    reviews(first: 10) { author rating body }
    recommendations(first: 5) { id name price thumbnail }
  }
}
```

The GraphQL server (the BFF) resolves each field by calling the appropriate downstream services. Facebook (who invented GraphQL) built it precisely for this use case — serving the same domain data efficiently across their mobile and web clients.

### Apollo Federation (Distributed GraphQL BFF)

```graphql
# Services define their schema pieces
# Product Service
type Product @key(fields: "id") {
  id: ID!
  name: String!
  price: Float!
}

# Review Service — extends Product
extend type Product @key(fields: "id") {
  reviews: [Review!]!  # added by Review Service
}

# Apollo Gateway federates into single BFF schema
# Clients query one endpoint; gateway fans out to services
```

Netflix, Expedia, and The New York Times use Apollo Federation as their BFF layer across 50+ downstream services.

## BFF for Different Protocols

```
Mobile BFF:     REST/JSON + aggressive HTTP caching + CDN
Web BFF:        REST/JSON or GraphQL + WebSockets for live updates
TV/Set-top BFF: REST/JSON with pre-rendered HTML snippets
Partner API:    REST with OpenAPI spec, versioning, API keys
Internal tools: gRPC for efficiency, no public exposure needed
```

## Real-World Implementations

**Netflix**: Maintains separate BFFs for TV, mobile (iOS/Android), web browser, and gaming consoles. Each BFF team sits with the client team — the mobile BFF team is part of the mobile engineering org.

```
Netflix BFF layers:
  TV BFF: Optimized for 60fps rendering, large image assets, linear navigation
  Mobile BFF: Compressed payloads, offline sync, battery-aware API patterns
  Web BFF: Rich metadata, PWA features, detailed reporting data
```

**SoundCloud**: One of the earliest BFF adopters. Separate API services for iOS app, Android app, and web — each evolved independently as client needs diverged.

**Spotify**: Their "Backstage" internal platform evolved from BFF principles — each client platform team owns their aggregation layer. The Web Player BFF handles WebSocket-based real-time sync differently than the mobile BFF.

**Zalando (E-commerce)**: Separate BFF for their mobile apps and desktop web. Mobile BFF returns product tiles with 3 fields; web BFF returns 30+ fields per product.

## Auth at the BFF Layer

BFFs are an excellent place to handle client-specific auth flows:

```typescript
// Mobile BFF: OAuth PKCE flow
// Web BFF: Cookie-based session + CSRF protection
// Partner API: API key + rate limiting

// Mobile BFF auth middleware
app.use('/api/mobile', async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  const user = await jwtService.verify(token);  // mobile uses JWT
  req.user = user;
  next();
});

// Web BFF auth middleware
app.use('/api/web', async (req, res, next) => {
  const sessionId = req.cookies['session'];
  const user = await sessionStore.get(sessionId);  // web uses sessions
  if (!user) return res.redirect('/login');
  req.user = user;
  next();
});
```

## BFF vs. API Gateway

| Aspect | API Gateway | BFF |
|--------|-------------|-----|
| Purpose | Cross-cutting concerns (auth, rate limiting, routing) | Client-specific data aggregation and shaping |
| Who writes it | Platform/infrastructure team | Frontend/mobile team |
| Business logic | None | Yes — aggregation, transformation |
| Number of instances | One (or one per zone) | One per client type |
| Protocol translation | Yes (sometimes) | Yes |

They compose: traffic flows through the API Gateway → BFF → downstream services.

## Trade-offs

| Pros | Cons |
|------|------|
| Optimized responses per client — less over/under fetching | More services to maintain and deploy |
| Client teams own their own backend — no cross-team dependencies | Risk of logic duplication across BFFs |
| Independent evolution per client | Shared downstream still constrained by slowest service |
| Right authentication model per client | Requires discipline to avoid BFFs becoming mini-monoliths |
| Reduces client-side data processing burden | N BFFs × M services = N×M integration points to manage |

## Anti-Patterns

- **Business logic in BFF**: BFF aggregates and shapes — it should NOT contain core business rules; those belong in domain services
- **Too many BFFs**: One per user or one per feature is overkill; one per genuinely different client type
- **BFF as the data source of truth**: BFF reads from services; it should never be the primary data store
- **Ignoring the BFF when services evolve**: Downstream service changes must be coordinated with BFF teams

## When to Use

✅ **Use when**: Multiple client types with genuinely different data needs; frontend teams blocked waiting for a shared API team; mobile clients needing bandwidth optimization; different auth flows per client type.

❌ **Avoid when**: Single client type; simple APIs where one response shape works for all consumers; small teams where maintaining multiple BFFs is too expensive.

---
*Related: [[API Gateway Pattern]], [[Clean Architecture]], [[Serverless Architecture]], [[Event-Driven Architecture]]*
