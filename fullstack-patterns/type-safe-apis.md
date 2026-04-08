---
title: "Type-Safe APIs"
category: fullstack-patterns
summary: "End-to-end type safety between client and server using shared TypeScript types, eliminating a class of runtime errors and enabling fearless refactoring across the full stack."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Type-Safe APIs

> End-to-end type safety between client and server using shared TypeScript types, eliminating a class of runtime errors and enabling fearless refactoring across the full stack.

## What It Is

Type-safe APIs share TypeScript type definitions between the frontend and backend so that API contracts are enforced at compile time. Change a response shape on the server — the TypeScript compiler immediately flags all broken client usages. No more runtime `Cannot read property 'x' of undefined`.

## tRPC — Type Safety Without Code Generation

tRPC is the most elegant solution: the router definition *is* the type. No schemas, no code gen, just TypeScript inference.

```typescript
// server/router.ts
import { initTRPC } from '@trpc/server';
import { z } from 'zod';

const t = initTRPC.context<Context>().create();
const router = t.router;
const publicProcedure = t.procedure;
const protectedProcedure = t.procedure.use(authMiddleware);

export const appRouter = router({
  users: router({
    getById: publicProcedure
      .input(z.object({ id: z.string().uuid() }))
      .query(async ({ input, ctx }) => {
        return ctx.db.user.findUniqueOrThrow({ where: { id: input.id } });
      }),
    
    create: protectedProcedure
      .input(z.object({
        name: z.string().min(1).max(100),
        email: z.string().email(),
      }))
      .mutation(async ({ input, ctx }) => {
        return ctx.db.user.create({ data: input });
      }),
  }),
  
  posts: router({
    list: publicProcedure
      .input(z.object({ cursor: z.string().optional(), limit: z.number().min(1).max(100).default(20) }))
      .query(async ({ input, ctx }) => {
        // Return type inferred from Prisma — flows through to client
        return ctx.db.post.findMany({ take: input.limit });
      }),
  }),
});

export type AppRouter = typeof appRouter;
```

```typescript
// client/api.ts — NO code generation required
import { createTRPCReact } from '@trpc/react-query';
import type { AppRouter } from '../server/router';

export const trpc = createTRPCReact<AppRouter>();

// Usage in component — fully typed, IDE autocomplete works
function UserProfile({ userId }: { userId: string }) {
  const { data, isLoading } = trpc.users.getById.useQuery({ id: userId });
  // data is typed as: { id: string; name: string; email: string; ... }
  
  const createUser = trpc.users.create.useMutation();
  // createUser.mutate({ name: 'Alice', email: 'alice@example.com' })
  // TypeScript error if you pass wrong shape ^^^
}
```

**tRPC + Next.js App Router**:
```typescript
// app/api/trpc/[trpc]/route.ts
import { fetchRequestHandler } from '@trpc/server/adapters/fetch';
import { appRouter } from '@/server/router';

const handler = (req: Request) =>
  fetchRequestHandler({ endpoint: '/api/trpc', req, router: appRouter, createContext });

export { handler as GET, handler as POST };
```

## Zod for Runtime Validation

tRPC uses Zod for input validation. The same schema validates at runtime AND provides TypeScript types — no duplication:

```typescript
const CreateUserSchema = z.object({
  name: z.string().min(1, 'Name required').max(100),
  email: z.string().email('Invalid email'),
  age: z.number().int().min(18).optional(),
  role: z.enum(['admin', 'user', 'viewer']).default('user'),
});

type CreateUserInput = z.infer<typeof CreateUserSchema>;
// { name: string; email: string; age?: number | undefined; role: 'admin' | 'user' | 'viewer' }

// Validate at runtime
const result = CreateUserSchema.safeParse(requestBody);
if (!result.success) {
  return { errors: result.error.flatten() };
}
```

## Zodios — REST with Zod Schemas

Zodios provides type-safe REST API clients using Zod schemas for contract-first development:

```typescript
import { makeApi, Zodios } from '@zodios/core';

const api = makeApi([
  {
    method: 'get',
    path: '/users/:id',
    alias: 'getUser',
    response: z.object({ id: z.string(), name: z.string(), email: z.string() }),
  },
  {
    method: 'post',
    path: '/users',
    alias: 'createUser',
    parameters: [{ name: 'body', type: 'Body', schema: CreateUserSchema }],
    response: z.object({ id: z.string() }),
  },
]);

const client = new Zodios('https://api.example.com', api);
const user = await client.getUser({ params: { id: '123' } });
// user is typed as { id: string; name: string; email: string }
```

## Contract-First with OpenAPI + TypeScript

For REST APIs that need to be public, generate TypeScript types from OpenAPI specs:

```yaml
# openapi.yaml
paths:
  /users/{id}:
    get:
      responses:
        '200':
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/User'
```

```bash
# Generate TypeScript types from OpenAPI
npx openapi-typescript openapi.yaml -o src/types/api.d.ts
```

```typescript
// Use generated types
import type { paths } from './types/api';
type User = paths['/users/{id}']['get']['responses']['200']['content']['application/json'];
```

**openapi-fetch** (from openapi-typescript team): fully typed fetch client with zero runtime overhead:
```typescript
import createClient from 'openapi-fetch';
const client = createClient<paths>({ baseUrl: 'https://api.example.com' });
const { data, error } = await client.GET('/users/{id}', { params: { path: { id: '123' } } });
```

## Comparison: tRPC vs. REST+OpenAPI vs. GraphQL

| | tRPC | REST + OpenAPI | GraphQL |
|--|------|----------------|---------|
| **Setup** | Minimal | Medium (schema maintenance) | High |
| **Type safety** | Compile-time, no gen | Generated from spec | Generated from schema |
| **Public API** | ❌ Not suitable | ✅ Industry standard | ✅ Introspectable |
| **Caching** | React Query built-in | HTTP cache native | Apollo cache |
| **Non-TS clients** | ❌ | ✅ Any language | ✅ Any language |
| **Migrations** | Rename and TypeScript errors | Manual versioning | Deprecate fields |
| **Best for** | Full-stack TypeScript monorepo | Multi-client public API | Complex data graph |

## Middleware and Context

```typescript
// tRPC context with auth and DB
const createContext = async ({ req }: CreateNextContextOptions) => {
  const token = req.headers.authorization?.replace('Bearer ', '');
  const user = token ? await verifyJWT(token) : null;
  return { db: prisma, user };
};

// Type-safe middleware — narrows context
const authMiddleware = t.middleware(({ ctx, next }) => {
  if (!ctx.user) throw new TRPCError({ code: 'UNAUTHORIZED' });
  return next({ ctx: { ...ctx, user: ctx.user } }); // user is non-null here
});
```

## Trade-offs

| Pro | Con |
|-----|-----|
| Compile-time API contract checking | tRPC only works in TypeScript ecosystems |
| Fearless refactoring across stack | Not suitable for public/third-party APIs |
| No schema duplication (tRPC) | Learning curve for teams used to REST |
| IDE autocomplete on API calls | Bundle size concerns (mitigated with server components) |
| Runtime validation prevents bad data | Requires monorepo or shared package |

## When to Use

✅ Full-stack TypeScript teams (Next.js, SvelteKit, Remix)  
✅ Internal APIs consumed only by your own frontend  
✅ Teams that have been burned by API contract drift  
✅ Monorepo setups where types can be shared directly  
❌ APIs consumed by non-TypeScript clients  
❌ Microservices needing language-agnostic contracts (use OpenAPI/Protobuf)  
❌ Public developer APIs where discoverability matters

---
*Related: [[GraphQL Architecture]], [[Monorepo Architecture]], [[Incremental Adoption Patterns]]*
