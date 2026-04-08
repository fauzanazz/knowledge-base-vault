---
title: "Incremental Adoption Patterns"
category: fullstack-patterns
summary: "Strategies for migrating legacy frontend and backend systems without big-bang rewrites, using the strangler fig pattern, codemods, and compatibility layers to incrementally replace components."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Incremental Adoption Patterns

> Strategies for migrating legacy frontend and backend systems without big-bang rewrites, using the strangler fig pattern, codemods, and compatibility layers to incrementally replace components.

## Why Incremental Migration

Big-bang rewrites fail at a well-documented rate (~80%). Teams underestimate complexity, business logic is lost in translation, and the product stagnates for 12–18 months. The strangler fig pattern (Martin Fowler) is the proven alternative: new system grows alongside the old, gradually taking over until the old one can be removed.

## The Strangler Fig Pattern

Named after a fig tree that grows around a host tree until the host dies:

```
Phase 1:  [Legacy App] (100% of traffic)

Phase 2:  [Proxy/Router] → [Legacy App] (95%)
                         → [New App]    (5%, new features)

Phase 3:  [Proxy/Router] → [Legacy App] (30%, remaining pages)
                         → [New App]    (70%, migrated pages)

Phase 4:  [New App] (100%)
          [Legacy App] → decommissioned
```

## Frontend Strangler Fig: Next.js + Legacy App

A common production pattern: migrating from a legacy CRA/Rails/PHP app to Next.js page by page.

```nginx
# Nginx reverse proxy — route by URL pattern
server {
    location / {
        # New Next.js app handles most routes
        proxy_pass http://nextjs:3000;
    }
    
    location /legacy/ {
        # Legacy app still handles unmigrated sections
        proxy_pass http://legacy-app:4000;
    }
    
    location /checkout {
        # Legacy checkout still running
        proxy_pass http://legacy-app:4000;
    }
}
```

**Vercel rewrites for Next.js**:
```json
// next.config.js
{
  "rewrites": [
    {
      "source": "/old-section/:path*",
      "destination": "https://legacy.internal.example.com/old-section/:path*"
    }
  ]
}
```

This lets Next.js own the domain while proxying unmigrated paths to legacy.

## Micro-Frontend Architecture as Migration Tool

Embed new React components inside legacy pages (opposite of strangler fig — gut and replace per component):

```typescript
// Legacy page (jQuery/vanilla JS)
// New React island embedded via mount point
document.addEventListener('DOMContentLoaded', () => {
  const mountPoint = document.getElementById('react-checkout');
  if (mountPoint) {
    const root = createRoot(mountPoint);
    root.render(<CheckoutWidget orderId={mountPoint.dataset.orderId} />);
  }
});
```

**Module Federation** (Webpack 5): share React components between legacy and new apps without duplication:
```javascript
// Legacy app consumes component from new app at runtime
const NewCheckout = React.lazy(() => import('newApp/Checkout'));
```

## Codemods for Large-Scale Refactors

Codemods are AST-based transforms that update code automatically. Essential for large-scale migrations:

### jscodeshift (JavaScript/TypeScript)
```typescript
// codemod: migrate from React.FC to function declaration
import { Transform } from 'jscodeshift';

const transform: Transform = (file, api) => {
  const j = api.jscodeshift;
  return j(file.source)
    .find(j.VariableDeclaration)
    .filter(path => {
      // Find: const MyComp: React.FC = () => ...
      return path.node.declarations.some(d => 
        d.id?.typeAnnotation?.typeAnnotation?.typeName?.name === 'FC'
      );
    })
    .replaceWith(path => {
      // Replace with: function MyComp() { ... }
      // ... transformation logic
    })
    .toSource();
};

export default transform;
```

```bash
# Apply codemod to entire codebase
npx jscodeshift -t codemod-fc-to-function.ts src/**/*.tsx --dry  # preview
npx jscodeshift -t codemod-fc-to-function.ts src/**/*.tsx       # apply
```

### ts-morph (TypeScript-native)
```typescript
import { Project } from 'ts-morph';

const project = new Project({ tsConfigFilePath: 'tsconfig.json' });

// Find all usages of deprecated API and replace
project.getSourceFiles().forEach(file => {
  file.getImportDeclarations()
    .filter(imp => imp.getModuleSpecifierValue() === 'old-package')
    .forEach(imp => imp.setModuleSpecifier('new-package'));
});

project.saveSync();
```

### Official Framework Codemods
```bash
# Next.js migration codemods
npx @next/codemod cra-to-next        # Create React App → Next.js
npx @next/codemod next-async-request-api .  # Pages → App Router
npx @next/codemod built-in-next-link .

# React 18 codemods
npx react-codemod rename-unsafe-lifecycles src/
```

## Compatibility Layers

### API Compatibility Shims
When migrating APIs, maintain the old contract while the new one stabilizes:

```typescript
// Legacy endpoint maintained as compatibility shim
// Routes to new internal service
app.get('/api/v1/users/:id', async (req, res) => {
  // Old format expected by legacy clients
  const user = await newUserService.getUser(req.params.id);
  
  // Transform new response to old schema
  res.json({
    user_id: user.id,          // was snake_case
    user_name: user.name,       // renamed fields
    email_address: user.email,  // renamed fields
  });
});
```

### Database Dual-Write Pattern
When migrating to a new data store:
```typescript
async function createUser(data: CreateUserDTO) {
  // Write to both old and new systems during migration
  const [oldUser, newUser] = await Promise.all([
    legacyDb.users.insert(data),
    newDb.user.create({ data }),
  ]);
  
  // Verify consistency
  if (oldUser.id !== newUser.id) {
    logger.error('Consistency violation during migration', { oldUser, newUser });
  }
  
  return newUser;
}
```

## Feature Flag-Driven Migration

Use feature flags to switch traffic between old and new implementations:

```typescript
async function getUserProfile(userId: string) {
  const useNewService = await flags.isEnabled('new-profile-service', { userId });
  
  if (useNewService) {
    return newProfileService.get(userId);    // new path
  }
  return legacyProfileService.get(userId);  // old path
}
```

**Shadow mode**: run both, compare results, use old result in production:
```typescript
async function getUserProfile(userId: string) {
  const [legacy, newResult] = await Promise.allSettled([
    legacyProfileService.get(userId),
    newProfileService.get(userId),
  ]);
  
  // Log discrepancies without affecting users
  if (legacy.status === 'fulfilled' && newResult.status === 'fulfilled') {
    if (!deepEqual(legacy.value, newResult.value)) {
      logger.warn('Shadow mode divergence', { userId, legacy: legacy.value, new: newResult.value });
    }
  }
  
  return legacy.value;  // Always use legacy until confident
}
```

## Migration Readiness Checklist

Before migrating each section:
- [ ] Coverage: new implementation has feature parity
- [ ] Tested: E2E tests cover migrated functionality  
- [ ] Rollback: flag or proxy can revert in <5 minutes
- [ ] Monitoring: alerts on new implementation errors
- [ ] Documentation: updated for new patterns

## Trade-offs

| Pro | Con |
|-----|-----|
| No big-bang risk | Two systems to maintain during migration |
| Features ship during migration | Complexity of routing/proxy layer |
| Rollback always available | Dual-write increases latency temporarily |
| Teams learn new patterns incrementally | Consistency management is hard |

## When to Use

✅ Any legacy system rewrite (frontend or backend)  
✅ Framework migrations (CRA→Next.js, Angular→React, Monolith→Microservices)  
✅ Database migrations with zero downtime requirement  
✅ API versioning when breaking changes are unavoidable  
❌ Simple, low-risk changes (over-engineering)  
❌ Systems with no test coverage (codemods and compatibility layers need tests)

---
*Related: [[Feature Flags]], [[Monorepo Architecture]], [[Type-Safe APIs]]*
