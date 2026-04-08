---
title: "GraphQL Architecture"
category: fullstack-patterns
summary: "A query language and runtime for APIs enabling clients to request exactly the data they need, with patterns for schema design, performance optimization via DataLoader, and scaling through federation."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# GraphQL Architecture

> A query language and runtime for APIs enabling clients to request exactly the data they need, with patterns for schema design, performance optimization via DataLoader, and scaling through federation.

## What It Is

GraphQL defines a typed schema for your API and lets clients specify exactly what fields they need — no over-fetching, no under-fetching. One endpoint replaces dozens of REST endpoints. Backed by a resolver tree that maps schema fields to data sources.

## Schema Design

### Schema-First vs. Code-First

**Schema-first** (SDL): write `.graphql` files, generate resolvers:
```graphql
type Query {
  user(id: ID!): User
  posts(filter: PostFilter, limit: Int = 20, cursor: String): PostConnection
}

type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post!]!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
}

type PostEdge {
  node: Post!
  cursor: String!
}
```

**Code-first** (Pothos, TypeGraphQL): generate schema from TypeScript, safer with refactoring:
```typescript
// Pothos (recommended for TypeScript)
const UserType = builder.objectRef<User>('User');
UserType.implement({
  fields: (t) => ({
    id: t.exposeID('id'),
    name: t.exposeString('name'),
    posts: t.field({
      type: [PostType],
      resolve: (user, _, ctx) => ctx.loaders.postsByUserId.load(user.id),
    }),
  }),
});
```

## Resolvers

Resolvers are functions that return data for each field. The key insight: resolvers form a tree that mirrors the schema.

```typescript
const resolvers = {
  Query: {
    user: async (_, { id }, ctx) => ctx.db.user.findUnique({ where: { id } }),
    posts: async (_, { filter, limit, cursor }, ctx) => {
      return ctx.db.post.findMany({
        where: filter,
        take: limit,
        cursor: cursor ? { id: cursor } : undefined,
      });
    },
  },
  User: {
    // This resolver runs once PER user in the response — N+1 problem!
    posts: async (user, _, ctx) => ctx.db.post.findMany({ where: { userId: user.id } }),
  },
};
```

## DataLoader (Solving N+1)

The N+1 problem: fetching 100 users fires 100 separate `posts` queries. DataLoader batches them into one:

```typescript
import DataLoader from 'dataloader';

// Create per-request loaders (NOT singleton — request-scoped)
function createLoaders(db: PrismaClient) {
  return {
    postsByUserId: new DataLoader<string, Post[]>(async (userIds) => {
      const posts = await db.post.findMany({
        where: { userId: { in: userIds as string[] } },
      });
      // Group by userId and return in same order as input
      return userIds.map(id => posts.filter(p => p.userId === id));
    }),
    
    userById: new DataLoader<string, User | null>(async (ids) => {
      const users = await db.user.findMany({ where: { id: { in: ids as string[] } } });
      const map = new Map(users.map(u => [u.id, u]));
      return ids.map(id => map.get(id) ?? null);
    }),
  };
}

// Context factory — loaders are per-request
const context = ({ req }) => ({
  db,
  loaders: createLoaders(db),
  user: authenticate(req),
});
```

Result: 100 users → 1 batched SQL query instead of 100.

## Pagination

**Cursor-based pagination** (recommended for feeds, infinite scroll):
```graphql
query {
  posts(limit: 20, cursor: "eyJpZCI6IjEwMCJ9") {
    edges {
      node { id title }
      cursor
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

**Offset pagination**: simpler but breaks with real-time inserts. Use only for stable datasets.

## GraphQL Federation

Federation splits a single schema across multiple services — each team owns their portion:

```typescript
// User Service — defines User type
const typeDefs = gql`
  extend schema @link(url: "https://specs.apollo.dev/federation/v2.0", import: ["@key"])
  
  type User @key(fields: "id") {
    id: ID!
    name: String!
    email: String!
  }
  
  type Query {
    user(id: ID!): User
  }
`;

// Posts Service — extends User from another service
const typeDefs = gql`
  type User @key(fields: "id") {
    id: ID!
    posts: [Post!]!   # Adds posts field to User without modifying User service
  }
  
  type Post {
    id: ID!
    title: String!
    author: User!
  }
`;
```

**Apollo Router** (Rust-based) handles query planning and stitching at the gateway level. Far faster than the JavaScript gateway.

## Subscriptions

Real-time GraphQL with WebSockets:

```typescript
// Server (graphql-ws library — recommended over subscriptions-transport-ws)
import { useServer } from 'graphql-ws/lib/use/ws';

const typeDefs = gql`
  type Subscription {
    messageAdded(channelId: ID!): Message!
  }
`;

const resolvers = {
  Subscription: {
    messageAdded: {
      subscribe: (_, { channelId }, ctx) => {
        return pubsub.asyncIterator(`CHANNEL_${channelId}`);
      },
    },
  },
};

// Publish from mutation resolver
const resolvers = {
  Mutation: {
    sendMessage: async (_, args, ctx) => {
      const message = await db.message.create({ data: args });
      await pubsub.publish(`CHANNEL_${args.channelId}`, { messageAdded: message });
      return message;
    },
  },
};
```

**PubSub scaling**: in-memory pubsub works for single-instance only. Use Redis pubsub adapter for multi-instance.

## Persisted Queries

In production, send a hash instead of the full query string — reduces payload size and prevents arbitrary query execution:

```typescript
// Client sends hash
{ "extensions": { "persistedQuery": { "version": 1, "sha256Hash": "abc123..." } } }

// Server looks up query by hash
const KNOWN_QUERIES = new Map([['abc123...', FULL_QUERY_STRING]]);
```

**Automatic Persisted Queries (APQ)**: Apollo client sends hash first; on cache miss, falls back to full query and server caches it.

## Security

```typescript
// Depth limiting — prevent deeply nested queries
import depthLimit from 'graphql-depth-limit';
const server = new ApolloServer({
  validationRules: [depthLimit(7)],
});

// Query complexity — prevent expensive queries
import { createComplexityLimitRule } from 'graphql-validation-complexity';
const server = new ApolloServer({
  validationRules: [createComplexityLimitRule(1000)],
});

// Disable introspection in production
const server = new ApolloServer({
  introspection: process.env.NODE_ENV !== 'production',
});
```

## Trade-offs

| Pro | Con |
|-----|-----|
| No over/under-fetching | Caching harder (single endpoint vs. per-URL) |
| Strongly typed schema | N+1 requires DataLoader discipline |
| Self-documenting API | Learning curve for new developers |
| Federation for microservices | Query complexity attacks require defense |
| Subscriptions built-in | File uploads need workarounds (multipart) |

## When to Use

✅ Multiple clients (web/mobile) with different data needs  
✅ Microservices needing unified API gateway  
✅ Rapid product iteration (add fields without versioning)  
✅ Collaborative features requiring real-time subscriptions  
❌ Simple CRUD APIs (REST is simpler)  
❌ File uploads as primary use case  
❌ Public APIs (REST with OpenAPI has better tooling)  
❌ Extremely latency-sensitive (REST has less parsing overhead)

---
*Related: [[Type-Safe APIs]], [[WebSocket Architecture]], [[Observability Stack]]*
