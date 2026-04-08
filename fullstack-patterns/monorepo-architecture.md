---
title: "Monorepo Architecture"
category: fullstack-patterns
summary: "A single repository containing multiple projects with shared tooling, build caching, and dependency management to maximize code reuse and developer velocity."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Monorepo Architecture

> A single repository containing multiple projects with shared tooling, build caching, and dependency management to maximize code reuse and developer velocity.

## What It Is

A monorepo (monolithic repository) stores multiple apps, libraries, and services in one VCS repository. Unlike a polyrepo approach, every package is versioned together, shares tooling config, and can reference each other directly without publishing to a registry.

Major adopters: Google (Blaze/Bazel), Meta (Buck), Twitter, Airbnb, Vercel (Turborepo internally), Microsoft (Nx).

## Core Tools

### Turborepo
- **Best for**: JavaScript/TypeScript projects with Next.js, Remix, or similar
- Remote caching via Vercel or self-hosted (e.g., `turbo run build --cache-dir=.turbo`)
- Task graph defined in `turbo.json`; parallel execution with dependency awareness
- Zero-config for most setups; `turbo.json` `pipeline` defines inputs/outputs per task

```json
// turbo.json
{
  "pipeline": {
    "build": { "dependsOn": ["^build"], "outputs": [".next/**", "dist/**"] },
    "lint": { "outputs": [] },
    "test": { "dependsOn": ["build"], "inputs": ["src/**/*.ts", "**/*.test.ts"] }
  }
}
```

### Nx
- **Best for**: Enterprise-scale, polyglot (TS, Go, Python, Java), plugin ecosystem
- Computation caching + distributed task execution (Nx Cloud)
- Affected graph: `nx affected:test --base=main` runs only impacted tests
- Generators/schematics for scaffolding new packages

### Bazel
- **Best for**: Very large codebases, cross-language (C++, Java, Go, Python)
- Hermetic builds: inputs/outputs are fully declared; reproducible anywhere
- Steep learning curve; BUILD files can be verbose
- Used at Google, Stripe, Dropbox at scale

## Dependency Management

**Internal packages**: Use workspace protocols (`"@company/ui": "workspace:*"`) to reference local packages. pnpm workspaces and npm/yarn workspaces support this natively.

**Version strategies**:
- **Fixed versioning** (Lerna + Nx): all packages share one version number
- **Independent versioning**: packages release separately (better for public libs)
- **No versioning**: internal-only packages never publish; just share by path

**Phantom dependencies**: Hoist carefully. pnpm's strict linking prevents accidental access to undeclared deps; prefer `shamefully-hoist=false`.

## Build Caching

Local cache: task outputs (e.g., `dist/`, `.next/`) stored by input hash. Cache hit = skip execution.

Remote cache: shared across CI and developers. Turborepo supports S3-compatible storage or Vercel's hosted cache. Nx Cloud offers distributed task execution (tasks run in parallel across agents).

```bash
# Turborepo with remote cache
TURBO_TOKEN=xxx TURBO_TEAM=myteam turbo run build
# Cache hit rate 80-95% in mature monorepos
```

## Trade-offs

| Pro | Con |
|-----|-----|
| Atomic cross-package changes | Git history gets large; `git clone` is slow |
| Shared lint/TS/test config | Initial setup overhead |
| Easier refactoring across packages | IDE performance can degrade |
| Single CI pipeline (with caching) | Ownership/access control is harder |
| Versioning coherence | Requires discipline to avoid tight coupling |

## Repo Structure (Recommended)

```
apps/
  web/          # Next.js app
  api/          # Express/Fastify service
  mobile/       # React Native
packages/
  ui/           # Shared component library
  config/       # ESLint, TS, Tailwind configs
  utils/        # Shared utilities
  db/           # Prisma schema + client
tooling/
  eslint-config/
  tsconfig/
```

## When to Use

✅ Multiple apps sharing components, types, or business logic  
✅ Teams that do frequent cross-package refactors  
✅ Organizations wanting unified CI/CD and tooling standards  
❌ Completely independent products with no shared code  
❌ Teams without DevEx investment to set up and maintain tooling  
❌ Open-source projects where independent versioning is critical

## CI/CD Integration

Use affected commands to skip unrelated work:
```bash
# Only test what changed vs. main
npx nx affected:test --base=origin/main --head=HEAD
turbo run test --filter=...[origin/main]
```

GitHub Actions matrix builds can fan out per-package for parallelism.

---
*Related: [[Trunk-Based Development]], [[Feature Flags]], [[Type-Safe APIs]]*
