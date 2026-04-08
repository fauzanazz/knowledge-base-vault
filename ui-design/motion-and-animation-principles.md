---
title: "Motion & Animation Principles for UI"
category: ui-design
summary: "A comprehensive guide to motion in UI — Disney's 12 animation principles applied to interfaces, meaningful vs decorative motion, easing functions, CSS vs Web Animations API vs FLIP, accessibility, performance, micro-interactions, page transitions, loading states, and Framer Motion/GSAP patterns."
sources:
  - web-research
updated: 2026-04-08T18:25:00.000Z
---

# Motion & Animation Principles for UI

> A comprehensive guide to motion in UI — Disney's 12 animation principles applied to interfaces, meaningful vs decorative motion, easing functions, CSS vs Web Animations API vs FLIP, accessibility, performance, micro-interactions, page transitions, and library patterns.

---

## Disney's 12 Principles Applied to UI

Originally codified by Frank Thomas and Ollie Johnston in *The Illusion of Life* (1981), these principles describe how motion creates believability. Each maps directly to interface behavior.

| Principle | UI Application |
|-----------|---------------|
| **Squash & Stretch** | Buttons subtly scale on press; modals "breathe" slightly while loading |
| **Anticipation** | A delete button wobbles before removing an item; menus shift before opening |
| **Staging** | New content enters from a predictable direction; focus areas dim surroundings |
| **Straight-ahead / Pose-to-pose** | CSS keyframe animations are pose-to-pose; spring physics are straight-ahead |
| **Follow-through & Overlapping action** | List items stagger in; trailing elements settle after parent stops |
| **Slow-in / Slow-out** | `ease-in-out` — all UI transitions should avoid linear motion |
| **Arcs** | Tooltips and dropdowns travel along a slight arc rather than straight lines |
| **Secondary action** | Icon animates while a form submits; ripple on a button click |
| **Timing** | 100–200 ms for micro-interactions; 300–500 ms for layout transitions |
| **Exaggeration** | Subtle: error shake is slightly over-rotated to communicate urgency |
| **Solid drawing** | 3D transforms preserve perspective; `transform-style: preserve-3d` |
| **Appeal** | Spring physics feel natural; linear or overly rigid motion feels robotic |

**Rule of thumb**: UI animation durations should sit between **100 ms** (instant feedback) and **500 ms** (complex layout changes). Anything over 500 ms should be skippable or only used for onboarding.

---

## Meaningful Motion vs. Decorative Motion

The core question for every animation: **does it communicate something, or just look good?**

**Meaningful motion:**
- Conveys relationship — a card expanding into a detail view preserves spatial context
- Communicates state — a spinner says "work is happening"; a progress bar says "here's how much"
- Guides attention — an error field shakes to direct focus
- Reduces cognitive load — a panel sliding in from the right teaches "this is a sub-level"

**Decorative motion:**
- Hover sparkles, background particle effects, purely aesthetic transitions
- Acceptable as brand expression but should never delay interaction or consume attention

**Decision framework:**
1. Does removing this animation break comprehension? → Meaningful, keep it.
2. Does it delay an action the user needs to complete? → Remove or make instant.
3. Does it reinforce a mental model (hierarchy, location, cause/effect)? → Meaningful.
4. Is it only there because it looks cool? → Decorative; make it skippable and respect `prefers-reduced-motion`.

---

## Easing Functions

Linear motion looks mechanical. Real objects accelerate and decelerate — easing encodes physics.

```css
/* CSS keywords map to cubic-bezier values */
transition: transform 300ms ease;          /* ease: slow start, fast middle, slow end */
transition: transform 300ms ease-in;       /* accelerate into resting state */
transition: transform 300ms ease-out;      /* decelerate into resting state — most common */
transition: transform 300ms ease-in-out;   /* both sides, symmetrical — good for loops */
transition: transform 300ms linear;        /* constant — only for spinners/progress */

/* Custom cubic-bezier */
transition: transform 300ms cubic-bezier(0.34, 1.56, 0.64, 1); /* slight overshoot */
```

| Curve | Use Case |
|-------|----------|
| `ease-out` | Elements entering the screen (start fast, settle gently) |
| `ease-in` | Elements leaving the screen (start slow, exit quickly) |
| `ease-in-out` | Elements moving within the screen (smooth both ends) |
| `linear` | Spinners, progress bars, continuous animations |
| Custom overshoot | Playful UIs, confirmations, success states |

### Spring Physics

CSS cubic-bezier curves are fixed-duration. **Spring physics** are duration-independent — they resolve when energy dissipates — making them feel more natural, especially for gesture-driven UIs.

```js
// Framer Motion spring
animate={{ x: 0 }}
transition={{ type: "spring", stiffness: 300, damping: 20 }}

// CSS approximation using custom easing
transition: transform 500ms cubic-bezier(0.34, 1.56, 0.64, 1);
```

| Parameter | Effect |
|-----------|--------|
| `stiffness` (high) | Snappy, quick to settle |
| `damping` (low) | More bounce/oscillation |
| `mass` (high) | Heavy, slower response |
| `velocity` | Initial velocity from gesture (enables throw-to-dismiss) |

---

## Animation Implementation: CSS vs WAAPI vs FLIP

| Approach | Best For | Limitations |
|----------|----------|-------------|
| **CSS `transition`** | Simple state changes (hover, focus, toggle) | No fine-grained JS control; no sequencing |
| **CSS `@keyframes` + `animation`** | Looping, multi-step, self-contained effects | Hard to synchronize; JS interop is clunky |
| **Web Animations API (WAAPI)** | JS-driven, composable, performant imperative animation | Verbose; limited browser support for advanced features |
| **FLIP** | Layout transitions (reordering, expanding) | Requires measurement + coordination; manual setup |
| **Framer Motion / GSAP** | Complex orchestration, gestures, scroll-linked animation | Bundle cost; overkill for simple transitions |

### CSS Transitions & Animations

```css
/* Transition: single property change on state */
.button { transform: scale(1); transition: transform 150ms ease-out; }
.button:hover { transform: scale(1.05); }

/* Keyframe animation: looping / multi-step */
@keyframes pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.4; }
}
.skeleton { animation: pulse 1.5s ease-in-out infinite; }
```

### Web Animations API (WAAPI)

```js
// Imperative, cancellable, playback-controllable
const el = document.querySelector('.card');
const anim = el.animate(
  [{ transform: 'translateY(20px)', opacity: 0 },
   { transform: 'translateY(0)',    opacity: 1 }],
  { duration: 300, easing: 'ease-out', fill: 'forwards' }
);
anim.onfinish = () => console.log('done');
```

### FLIP (First, Last, Invert, Play)

FLIP enables smooth layout transitions by measuring before/after positions and animating the *difference*, keeping motion on the compositor thread.

```js
// 1. First: record current position
const first = el.getBoundingClientRect();

// 2. Last: apply new state (DOM reflow)
el.classList.add('expanded');
const last = el.getBoundingClientRect();

// 3. Invert: compute delta and apply as transform
const dx = first.left - last.left;
const dy = first.top  - last.top;
el.style.transform = `translate(${dx}px, ${dy}px)`;

// 4. Play: remove inversion and animate to natural position
requestAnimationFrame(() => {
  el.style.transition = 'transform 300ms ease-out';
  el.style.transform  = '';
});
```

Libraries like **Framer Motion** (`layoutId`) and **AutoAnimate** handle FLIP automatically.

---

## Reduced-Motion Accessibility

Users with vestibular disorders, epilepsy, or motion sensitivity can experience nausea or seizures from excessive animation. **Always respect `prefers-reduced-motion`.**

```css
/* CSS: disable or reduce motion globally */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

```js
// JS: conditionally animate
const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;

// Framer Motion: built-in hook
import { useReducedMotion } from 'framer-motion';
const reduce = useReducedMotion();
<motion.div animate={{ x: reduce ? 0 : 100 }} />
```

**Best practice**: Don't just skip animations — provide a *reduced* alternative. Fades and instant state changes are acceptable; spinning, parallax, and auto-playing video are not.

---

## Performance: Compositor-Only Properties

The browser rendering pipeline: **Style → Layout → Paint → Composite**. Triggering layout (reflow) is the most expensive step.

| Property | Triggers | Performance |
|----------|----------|-------------|
| `transform`, `opacity` | Composite only | ✅ 60 fps on GPU |
| `filter`, `clip-path` | Paint + Composite | ⚠️ Moderate |
| `width`, `height`, `top`, `left` | Layout + Paint + Composite | ❌ Causes reflow |
| `background-color`, `border` | Paint + Composite | ⚠️ Moderate |

**Rule**: Animate only `transform` and `opacity` whenever possible. Simulate `width` changes with `scaleX`.

### `will-change`

Promotes the element to its own compositor layer before animation begins, eliminating jank on first frame.

```css
.card:hover { will-change: transform; }              /* hint before animation */
.animating  { will-change: transform, opacity; }

/* ⚠️ Do NOT apply globally — each layer consumes GPU memory */
/* Remove after animation completes via JS */
el.addEventListener('transitionend', () => el.style.willChange = 'auto');
```

---

## Micro-Interactions

Micro-interactions are single-purpose, contained animations that confirm actions, communicate state, and create delight.

```css
/* Button press feedback */
.btn:active { transform: scale(0.96); transition: transform 80ms ease-in; }

/* Toggle switch */
.toggle-thumb { transition: transform 200ms cubic-bezier(0.34, 1.56, 0.64, 1); }
.toggle[checked] .toggle-thumb { transform: translateX(24px); }

/* Checkmark draw-on */
@keyframes draw {
  from { stroke-dashoffset: 100; }
  to   { stroke-dashoffset: 0;   }
}
.checkmark path { stroke-dasharray: 100; animation: draw 300ms ease-out forwards; }
```

---

## Page Transitions: View Transitions API

The **View Transitions API** (Chrome 111+, Safari 18+) enables seamless page transitions in SPAs and MPAs without a library.

```js
// Trigger a view transition
document.startViewTransition(() => {
  // DOM update — the browser captures before/after and animates
  updateDOM();
});
```

```css
/* Customize the default cross-fade */
::view-transition-old(root) { animation: 200ms ease-out slide-out; }
::view-transition-new(root) { animation: 200ms ease-in  slide-in;  }

/* Named elements persist across views (hero transitions) */
.product-card { view-transition-name: product-hero; }
```

For React Router / Next.js, wrappers like **next-view-transitions** and **react-router-view-transition** handle lifecycle integration.

---

## Loading States

| Pattern | When to Use | Implementation |
|---------|-------------|----------------|
| **Skeleton screens** | Content-heavy pages (feeds, cards) | CSS pulse animation on gray placeholders |
| **Spinner** | Short indeterminate waits (<3 s) | SVG rotating stroke; CSS `border-radius` spin |
| **Progress bar** | Determinate operations (file upload, step wizard) | `width` transition on a `<progress>` element |
| **Optimistic UI** | Low-latency mutations (like, bookmark, toggle) | Update immediately, revert on error |
| **Shimmer** | Alternative to skeleton; richer visual | Linear-gradient moving left-to-right via `backgroundPosition` |

```css
/* Shimmer loading effect */
.skeleton {
  background: linear-gradient(90deg, #e0e0e0 25%, #f5f5f5 50%, #e0e0e0 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}
@keyframes shimmer {
  from { background-position: 200% 0; }
  to   { background-position: -200% 0; }
}
```

---

## Framer Motion & GSAP Patterns

### Framer Motion (React)

```jsx
import { motion, AnimatePresence, stagger, useAnimate } from 'framer-motion';

// Mount/unmount transitions
<AnimatePresence>
  {isOpen && (
    <motion.div
      key="modal"
      initial={{ opacity: 0, y: 20 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: 20 }}
      transition={{ type: 'spring', stiffness: 400, damping: 30 }}
    />
  )}
</AnimatePresence>

// Layout animations (FLIP under the hood)
<motion.div layout layoutId="hero-card" />
```

### Stagger & Orchestration

```jsx
// Framer Motion: stagger children via variants
const container = {
  hidden: {},
  show: { transition: { staggerChildren: 0.07 } }
};
const item = {
  hidden: { opacity: 0, y: 16 },
  show:   { opacity: 1, y: 0, transition: { type: 'spring', stiffness: 300 } }
};

<motion.ul variants={container} initial="hidden" animate="show">
  {items.map(i => <motion.li key={i.id} variants={item} />)}
</motion.ul>
```

```js
// GSAP: timeline + stagger
import gsap from 'gsap';

gsap.timeline()
  .from('.hero-title',   { y: 40, opacity: 0, duration: 0.5, ease: 'power3.out' })
  .from('.hero-sub',     { y: 20, opacity: 0, duration: 0.4 }, '-=0.25')
  .from('.card',         { y: 20, opacity: 0, duration: 0.4, stagger: 0.08 }, '-=0.2');
```

| Library | Bundle | Strengths | Best For |
|---------|--------|-----------|----------|
| **Framer Motion** | ~45 KB gzip | React-native, layout animations, gestures, spring physics | React SPAs |
| **GSAP (free)** | ~23 KB gzip | Timeline control, ScrollTrigger, SVG, framework-agnostic | Marketing sites, complex scroll |
| **Motion One** | ~3 KB gzip | WAAPI wrapper, tiny, tree-shakeable | Performance-critical apps |
| **AutoAnimate** | ~2 KB gzip | Zero-config list/presence animations | Quick wins, no-fuss transitions |

---

## Quick-Reference: Duration Guidelines

| Interaction Type | Duration | Easing |
|-----------------|----------|--------|
| Micro (button press, hover) | 80–150 ms | `ease-out` |
| Element enter/exit | 200–300 ms | `ease-out` / spring |
| Panel / drawer slide | 250–350 ms | `ease-in-out` |
| Page transition | 300–500 ms | `ease-in-out` |
| Looping / idle animation | 1000–2000 ms | `ease-in-out` or `linear` |
| Onboarding / hero animation | 500–800 ms | Spring or custom |

---

## Trade-Offs at a Glance

| Decision | Prefer When | Avoid When |
|----------|-------------|------------|
| Spring physics | Gesture-driven, natural feel | Precise timing required (synced audio/video) |
| FLIP layout animation | Items reorder, expand/collapse | Simple opacity-only transitions (overkill) |
| View Transitions API | MPA or simple SPA transitions | Complex per-route orchestration needed |
| `will-change` | Pre-warming heavy animations | Global application (GPU memory waste) |
| GSAP over Framer Motion | Framework-agnostic or scroll animations | React-heavy codebase with gestures |
| Decorative animations | Brand moments, empty states | Core task flows, data-dense dashboards |
