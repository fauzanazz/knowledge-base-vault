---
title: "Client State Management"
category: frontend-architecture
summary: "A comprehensive guide to frontend state management paradigms — Flux/Redux, Atomic (Jotai/Recoil), Proxy-based (Valtio/MobX), URL state, and Server State (TanStack Query) — with trade-offs and selection criteria."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Client State Management

> A comprehensive guide to frontend state management paradigms — Flux/Redux, Atomic (Jotai/Recoil), Proxy-based (Valtio/MobX), URL state, and Server State (TanStack Query) — with trade-offs and selection criteria.

## Types of Client State

Before choosing a library, categorize your state:

| Type | Description | Best Solution |
|------|-------------|--------------|
| **Server state** | Data from APIs, async, cacheable | TanStack Query, SWR |
| **URL state** | Filters, pagination, tabs | URL params, router state |
| **UI state** | Modals, tooltips, local form state | useState, local component |
| **Global UI state** | Theme, auth, sidebar open | Zustand, Jotai, Context |
| **Form state** | Validation, dirty tracking | React Hook Form, Formik |

**Most state management problems are actually server state problems.** Using TanStack Query eliminates 60-70% of the need for global state stores.

---

## 1. Flux / Redux

**Flux** is Meta's unidirectional data flow pattern (2014). **Redux** (Dan Abramov, 2015) is its most popular implementation.

```
View → dispatch(Action) → Reducer → New State → View (re-render)
```

```ts
// Redux Toolkit (modern Redux)
import { createSlice, configureStore } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [], total: 0 },
  reducers: {
    addItem(state, action) {
      state.items.push(action.payload); // Immer — mutate safely
      state.total += action.payload.price;
    },
    removeItem(state, action) {
      state.items = state.items.filter(i => i.id !== action.payload);
    },
  },
});

const store = configureStore({ reducer: { cart: cartSlice.reducer } });
```

**Redux DevTools**: Time-travel debugging, action replay — unmatched DX for debugging complex flows.

**Used by**: Large enterprise apps, teams needing strict patterns, apps with complex business logic.

**Redux Toolkit (RTK)** is the modern API — eliminates boilerplate. **RTK Query** adds server state caching.

**Pros**: Predictable, debuggable, excellent DevTools, clear boundaries  
**Cons**: Verbose (even with RTK), over-engineered for small apps, boilerplate

---

## 2. Atomic State (Jotai / Recoil)

Atoms are independent units of state. Components subscribe to specific atoms — no selector overhead.

### Jotai (Daishi Kato, 2021)

```ts
import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Primitive atoms
const countAtom = atom(0);
const textAtom = atom('hello');

// Derived atom (computed)
const doubledAtom = atom((get) => get(countAtom) * 2);

// Async atom (replaces React Query for simple cases)
const userAtom = atom(async () => {
  const res = await fetch('/api/user');
  return res.json();
});

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const doubled = useAtomValue(doubledAtom);  // Read-only
  
  return <button onClick={() => setCount(c => c + 1)}>{count} ({doubled})</button>;
}
```

**Jotai advantages**: No Provider needed (uses WeakMap internally), tiny (3KB), works with React 18 Suspense and transitions, atoms are composable.

### Recoil (Meta, 2020)

Similar to Jotai but requires a `RecoilRoot` provider and was built by Meta internally:
```ts
const counterState = atom({ key: 'counter', default: 0 });
const doubledSelector = selector({
  key: 'doubled',
  get: ({ get }) => get(counterState) * 2,
});
```

**Note**: Recoil is in maintenance mode (Meta shifted focus). Jotai is the recommended successor.

**Used by**: Apps with many independent pieces of shared state, cross-cutting concerns.

---

## 3. Proxy-Based State (Valtio / MobX / Zustand)

### Zustand (Daishi Kato, 2019)

Simple, minimal store with a hook-based API. The most popular React state library in 2024.

```ts
import { create } from 'zustand';

interface CartStore {
  items: Item[];
  total: number;
  addItem: (item: Item) => void;
  removeItem: (id: string) => void;
}

const useCartStore = create<CartStore>((set, get) => ({
  items: [],
  total: 0,
  addItem: (item) => set(state => ({
    items: [...state.items, item],
    total: state.total + item.price,
  })),
  removeItem: (id) => set(state => ({
    items: state.items.filter(i => i.id !== id),
  })),
}));

function Cart() {
  const { items, addItem } = useCartStore();  // Only re-renders when items changes
  return <div>{items.length} items</div>;
}
```

**Zustand middleware**: `persist` (localStorage), `devtools` (Redux DevTools), `immer`.

### Valtio (Daishi Kato, 2021)

ES Proxy-based — mutate state directly, subscriptions auto-track:

```ts
import { proxy, useSnapshot } from 'valtio';

const state = proxy({ count: 0, name: 'Alice' });

// Mutate directly anywhere — no reducers needed
function increment() { state.count++; }

function Counter() {
  const snap = useSnapshot(state);  // Re-renders only when accessed values change
  return <div onClick={increment}>{snap.count}</div>;
}
```

### MobX

Mature proxy-based solution, class-oriented or functional:
```ts
import { makeAutoObservable } from 'mobx';

class CartStore {
  items: Item[] = [];
  
  constructor() { makeAutoObservable(this); }
  
  get total() { return this.items.reduce((sum, i) => sum + i.price, 0); }
  
  addItem(item: Item) { this.items.push(item); }  // Auto-tracked mutation
}
```

**Used by**: Enterprise apps with complex domain models, Angular codebases, teams from OOP backgrounds.

---

## 4. URL State

Often underused. URL state is shareable, bookmarkable, back-button-aware, and survives page refresh.

```ts
// Next.js App Router — URL search params as state
import { useSearchParams, useRouter, usePathname } from 'next/navigation';

function ProductFilters() {
  const searchParams = useSearchParams();
  const router = useRouter();
  const pathname = usePathname();
  
  const category = searchParams.get('category') ?? 'all';
  
  function setCategory(cat: string) {
    const params = new URLSearchParams(searchParams);
    params.set('category', cat);
    router.push(`${pathname}?${params.toString()}`);
  }
  
  return <select value={category} onChange={e => setCategory(e.target.value)}>...</select>;
}
```

**nuqs**: Type-safe URL state library for Next.js:
```ts
import { useQueryState, parseAsInteger } from 'nuqs';

const [page, setPage] = useQueryState('page', parseAsInteger.withDefault(1));
```

**Use URL state for**: Pagination, filters, sort order, tabs, search queries, modal IDs.

---

## 5. Server State (TanStack Query)

Server state is async, shared, and needs caching, deduplication, and revalidation. TanStack Query (formerly React Query) solves this:

```ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Automatic caching, deduplication, background refresh
function useProducts() {
  return useQuery({
    queryKey: ['products'],
    queryFn: () => fetch('/api/products').then(r => r.json()),
    staleTime: 5 * 60 * 1000,  // Fresh for 5 minutes
    gcTime: 10 * 60 * 1000,    // Keep in cache for 10 min
  });
}

// Optimistic updates
function useAddProduct() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (product) => fetch('/api/products', { method: 'POST', body: JSON.stringify(product) }),
    onMutate: async (product) => {
      await queryClient.cancelQueries({ queryKey: ['products'] });
      const previous = queryClient.getQueryData(['products']);
      queryClient.setQueryData(['products'], old => [...old, product]); // Optimistic
      return { previous };
    },
    onError: (err, _, ctx) => queryClient.setQueryData(['products'], ctx.previous),
    onSettled: () => queryClient.invalidateQueries({ queryKey: ['products'] }),
  });
}
```

**SWR** (Vercel, simpler): `useSWR('/api/data', fetcher)` with automatic revalidation.

**Used by**: Almost every modern React app. TanStack Query is the de facto server state solution.

---

## Choosing the Right Solution

```
Is it server/API data?
  └── Yes → TanStack Query or SWR

Is it navigation/filter state?
  └── Yes → URL params (nuqs for Next.js)

Is it local component state?
  └── Yes → useState or useReducer

Is it shared global UI state?
  ├── Simple (theme, auth, cart) → Zustand
  ├── Many independent atoms → Jotai
  ├── Complex business logic + debug needs → Redux Toolkit
  └── Mutable OOP-style → MobX or Valtio
```

## Bundle Size Comparison

| Library | Size (gzipped) |
|---------|---------------|
| Zustand | ~1KB |
| Jotai | ~3KB |
| Valtio | ~3KB |
| TanStack Query | ~13KB |
| Redux Toolkit | ~11KB |
| MobX | ~16KB |
| Recoil | ~21KB |

---
*Related: [[Signals and Fine-Grained Reactivity]], [[React Server Components]], [[Rendering Strategies]], [[SPA vs MPA vs Hybrid]]*
