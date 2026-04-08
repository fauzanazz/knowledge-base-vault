---
title: "Signals and Fine-Grained Reactivity"
category: frontend-architecture
summary: "A reactivity model where individual reactive values (signals) track their own subscriptions, enabling surgical UI updates that bypass virtual DOM diffing for near-zero overhead state changes."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Signals and Fine-Grained Reactivity

> A reactivity model where individual reactive values (signals) track their own subscriptions, enabling surgical UI updates that bypass virtual DOM diffing for near-zero overhead state changes.

## The Problem with Virtual DOM

React's model: state changes trigger component re-renders, then a VDOM diff determines what to update. Even with memoization, this is O(n) work proportional to component tree size.

```
React state change:
setState(newValue)
  → re-render entire component
    → compare VDOM with previous
      → patch only changed DOM nodes
```

For complex UIs with frequent updates (sliders, real-time data, animations), VDOM diffing is a bottleneck.

## What Are Signals?

A **signal** is a reactive primitive that wraps a value and tracks which computations depend on it. When the signal's value changes, only the subscribed computations re-run — no component re-render, no VDOM diff.

```
Signal state change:
signal.value = newValue
  → only DOM nodes directly bound to signal update
    → bypasses component tree entirely
```

The concept originates from **KnockoutJS** (2010), was popularized by **SolidJS** (2020), and has now spread to Angular, Preact, Vue, and (experimentally) React.

## SolidJS — The Pioneer

SolidJS compiles away the component abstraction: components run **once** to set up subscriptions, then never re-render. Only signals trigger updates.

```tsx
import { createSignal, createEffect, createMemo } from 'solid-js';

function Counter() {
  const [count, setCount] = createSignal(0);
  const doubled = createMemo(() => count() * 2); // Derived signal

  createEffect(() => {
    console.log('Count changed:', count()); // Runs only when count changes
  });

  // JSX compiles to direct DOM bindings, no VDOM
  return (
    <div>
      <p>Count: {count()}</p>
      <p>Doubled: {doubled()}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
// The <p> elements are bound DIRECTLY to count signal
// Clicking button updates only those DOM nodes — nothing else runs
```

**SolidJS benchmarks**: Consistently top 3 in [js-framework-benchmark](https://krausest.github.io/js-framework-benchmark/) — faster than React, Vue, Angular.

## Preact Signals

Preact introduced signals in v10.6 (2022) — usable in both Preact AND React:

```tsx
import { signal, computed, effect } from '@preact/signals-react'; // React adapter

const count = signal(0);
const doubled = computed(() => count.value * 2);

effect(() => console.log(count.value)); // Auto-subscribes

function Counter() {
  // Component only re-renders for non-signal props
  // count.value usage bypasses re-render — direct DOM update
  return (
    <div>
      <p>{count}</p>        {/* Directly bound — no re-render */}
      <p>{doubled}</p>      {/* Computed — no re-render */}
      <button onClick={() => count.value++}>+</button>
    </div>
  );
}
```

**Key innovation**: Signals can be passed *into* JSX directly — React bypasses reconciliation for signal-bound text nodes.

## Angular Signals (Angular 17+)

Angular's reactive system was always zone.js-based change detection (dirty checking entire component tree). Signals replace this:

```ts
import { signal, computed, effect } from '@angular/core';

@Component({
  template: `
    <p>Count: {{ count() }}</p>
    <p>Doubled: {{ doubled() }}</p>
    <button (click)="increment()">+</button>
  `
})
class CounterComponent {
  count = signal(0);
  doubled = computed(() => this.count() * 2);

  increment() { this.count.update(c => c + 1); }
}
```

Angular signals enable:
- **Zoneless Angular** (dropping zone.js — ~30KB saving)
- **OnPush by default** without manual optimization
- Tree-shakeable change detection

## Vue 3 Reactivity

Vue's reactivity has always been fine-grained (since Vue 2's `Object.defineProperty`). Vue 3 uses ES Proxies:

```ts
import { ref, computed, watchEffect } from 'vue';

const count = ref(0);           // Equivalent to a signal
const doubled = computed(() => count.value * 2); // Derived

watchEffect(() => {
  console.log(count.value);     // Auto-tracks dependencies
});
```

Vue's template compiler creates direct bindings to reactive data — similar efficiency to signals without calling them "signals."

## The Signals TC39 Proposal

A TC39 proposal (Stage 1) to standardize signals in JavaScript itself:

```js
// Proposed browser-native signals API
import { Signal } from 'signal-polyfill';

const counter = new Signal.State(0);
const doubled = new Signal.Computed(() => counter.get() * 2);

// Framework-agnostic — works with any renderer
counter.set(counter.get() + 1);
console.log(doubled.get()); // 2
```

Backed by Angular, Preact, Vue, Solid, Ember teams. If standardized, frameworks could share reactive infrastructure.

## Comparison Table

| Feature | React useState | Vue ref | SolidJS signal | Preact signal | Angular signal |
|---------|---------------|---------|----------------|---------------|----------------|
| Re-renders component | Yes | Yes (template) | No | Minimal | No |
| VDOM diffing | Yes | Yes | No | Reduced | No |
| Fine-grained DOM | No | Compiler | Yes | Yes | Yes |
| Async support | useTransition | watchEffect | createResource | async computed | toObservable |
| Learning curve | Low | Low | Medium | Low | Medium |
| Ecosystem maturity | Highest | High | Growing | Medium | High (Angular) |

## Performance Implications

- **SolidJS**: ~2-10x less DOM operations than React for frequent updates
- **Preact signals in React**: Up to 4x faster for signal-bound values
- **Angular zoneless**: ~40% reduction in change detection overhead
- **Vue 3 Proxy reactivity**: 2x faster than Vue 2 Object.defineProperty

## When to Use

✅ **Use signals when:**
- Frequent, fine-grained state updates (real-time, animations, forms)
- Performance is critical and VDOM overhead is measurable
- Already using SolidJS, Preact, Vue 3, or Angular 17+
- Want to avoid over-rendering without manual `useMemo`/`memo`

❌ **Stick with React model when:**
- Team expertise and ecosystem are React-primary
- App complexity doesn't warrant fine-grained optimization
- Using React without Preact signals adapter

---
*Related: [[Client State Management]], [[Rendering Strategies]], [[SPA vs MPA vs Hybrid]]*
