---
title: "Accessibility (WCAG 2.2)"
category: ui-design
summary: "A practical guide to web accessibility — WCAG 2.2 structure and conformance levels, POUR principles, new 2.2 criteria, semantic HTML, ARIA roles and live regions, focus management patterns, screen reader testing, color contrast (WCAG vs. APCA), keyboard navigation, form accessibility, reduced motion, and automated testing tools."
sources:
  - web-research
updated: 2026-04-08T18:27:00.000Z
---

# Accessibility (WCAG 2.2)

> A practical guide to web accessibility — WCAG 2.2 structure and conformance levels, POUR principles, new 2.2 criteria, semantic HTML, ARIA roles and live regions, focus management patterns, screen reader testing, color contrast (WCAG vs. APCA), keyboard navigation, form accessibility, reduced motion, and automated testing tools.

## WCAG 2.2 Structure & Conformance Levels

The Web Content Accessibility Guidelines (WCAG) 2.2 (published October 2023) organise all requirements into three conformance levels:

| Level | Meaning | Typical Target |
|-------|---------|---------------|
| **A** | Minimum — barriers that block entire user groups | Legal baseline; must pass |
| **AA** | Standard — addresses the majority of barriers | Industry standard; most legal mandates (ADA, EN 301 549) |
| **AAA** | Enhanced — narrow or complex scenarios | Specialist audiences (gov, healthcare) |

AA conformance requires satisfying **all A and AA criteria**. There are **9 guidelines** organised under 4 principles, containing **87 success criteria** in WCAG 2.2 (up from 78 in 2.1 — new criteria added at A and AA levels).

---

## POUR Principles

Every success criterion maps to one of four principles:

| Principle | Core Question | Key Criteria |
|-----------|--------------|-------------|
| **Perceivable** | Can users perceive all content? | Alt text (1.1.1), captions (1.2.2), contrast (1.4.3), reflow (1.4.10) |
| **Operable** | Can users operate all UI? | Keyboard (2.1.1), no seizures (2.3.1), focus visible (2.4.7), target size (2.5.8) |
| **Understandable** | Is content and UI understandable? | Language (3.1.1), labels (3.3.2), error identification (3.3.1) |
| **Robust** | Does content work with AT? | Name/Role/Value (4.1.2), status messages (4.1.3) |

---

## New in WCAG 2.2

WCAG 2.2 added six new success criteria. The three most impactful for UI engineers:

### 2.4.11 Focus Not Obscured — AA
A focused component must not be **entirely hidden** behind sticky headers, chat widgets, or cookie banners. At least part of the focused element must remain visible.

```css
/* Fix: ensure sticky headers don't cover focused elements */
:target, :focus {
  scroll-margin-top: 80px; /* height of sticky header */
}
```

### 2.5.7 Dragging Movements — AA
All functionality that uses a drag gesture must have a **single-pointer alternative** (e.g., a click-based fallback). Affects sortable lists, sliders, and map controls.

```tsx
// ✅ Provide keyboard/button alternatives alongside drag
<DraggableItem
  onDragEnd={handleReorder}
  onMoveUp={() => handleReorder('up')}   // button fallback
  onMoveDown={() => handleReorder('down')}
/>
```

### 2.5.8 Target Size (Minimum) — AA
Interactive targets must be at least **24×24 CSS pixels**. (AAA requires 44×44px.) Where targets are smaller, the space around them must add up to 24px with no overlap.

```css
/* Ensure tap targets meet minimum 24×24px */
.icon-button {
  min-width: 24px;
  min-height: 24px;
  padding: 10px; /* expands hit area toward the 44px recommended size */
}
```

---

## Semantic HTML

Native HTML elements carry built-in accessibility semantics — roles, states, keyboard behaviour, and name calculation — for free. Prefer them over `<div>` + ARIA.

```html
<!-- ❌ Custom widget — must manually add everything -->
<div role="button" tabindex="0" aria-pressed="false" onclick="..." onkeydown="...">
  Toggle
</div>

<!-- ✅ Native — role, keyboard, and state included -->
<button type="button" aria-pressed="false">Toggle</button>
```

| Native Element | Implicit Role | Notes |
|---------------|--------------|-------|
| `<button>` | `button` | Enter + Space activate; focusable by default |
| `<a href>` | `link` | Enter activates; no href = no role/focus |
| `<nav>` | `navigation` | Landmark; one per page unless labeled |
| `<main>` | `main` | One per page; skip link target |
| `<table>` | `table` | Requires `<caption>`, `<th scope>` for data tables |
| `<input type="checkbox">` | `checkbox` | Space toggles; pairs with `<label>` |

**Rule of thumb**: If a native element exists for your widget, use it. Only reach for ARIA when no native element fits.

---

## ARIA: Roles, States & Properties

ARIA **supplements** (never replaces) native semantics. The first rule of ARIA: *don't use ARIA if a native element can do the job.*

### Key Attributes

| Attribute | Type | Purpose | Example |
|-----------|------|---------|---------|
| `aria-label` | Property | Overrides accessible name | `<button aria-label="Close dialog">×</button>` |
| `aria-labelledby` | Property | Names element via another element's text | `<dialog aria-labelledby="title-id">` |
| `aria-describedby` | Property | Associates descriptive text | `<input aria-describedby="hint-id">` |
| `aria-expanded` | State | Open/closed state of a disclosure | `<button aria-expanded="true">` |
| `aria-haspopup` | Property | Indicates a popup (menu, listbox) | `<button aria-haspopup="listbox">` |
| `aria-hidden` | State | Removes element from AT tree | `<svg aria-hidden="true">` |
| `aria-disabled` | State | Communicates disabled without removing focus | `<button aria-disabled="true">` |
| `aria-live` | Property | Announces dynamic content (see Live Regions) | `<div aria-live="polite">` |
| `aria-busy` | State | Loading/updating in progress | `<div aria-busy="true">` |
| `aria-current` | State | Current item in a set (page, step, date) | `<a aria-current="page">` |
| `aria-controls` | Property | Points to element this widget controls | `<button aria-controls="panel-id">` |

### Live Regions

Live regions announce dynamic updates to screen readers without moving focus.

```html
<!-- Polite: waits for user to finish current activity (status messages) -->
<div role="status" aria-live="polite" aria-atomic="true">
  Form saved successfully.
</div>

<!-- Assertive: interrupts immediately (time-sensitive errors only) -->
<div role="alert" aria-live="assertive">
  Session expiring in 60 seconds.
</div>
```

| Setting | Behaviour | Use When |
|---------|-----------|----------|
| `aria-live="polite"` | Announces at next idle | Search results, save confirmations |
| `aria-live="assertive"` | Interrupts immediately | Critical errors, session warnings |
| `aria-atomic="true"` | Reads the whole region on change | Status message regions |
| `role="status"` | Polite live region shorthand | Toast notifications |
| `role="alert"` | Assertive live region shorthand | Error summaries |

**Gotcha**: Inject content *after* the live region is in the DOM, not simultaneously — screen readers must observe the region before announcements work.

---

## Focus Management

### Skip Links
Allow keyboard users to bypass repetitive navigation. Must be the **first focusable element** in the DOM; can be visually hidden until focused.

```html
<a href="#main-content" class="skip-link">Skip to main content</a>

<style>
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  transition: top 0.2s;
}
.skip-link:focus {
  top: 0;
}
</style>
```

### Focus Trapping (Modals & Dialogs)
When a modal opens, focus must be **trapped inside** it. When it closes, focus returns to the trigger.

```ts
function trapFocus(container: HTMLElement) {
  const focusable = container.querySelectorAll<HTMLElement>(
    'a[href], button:not([disabled]), input, select, textarea, [tabindex]:not([tabindex="-1"])'
  );
  const first = focusable[0];
  const last = focusable[focusable.length - 1];

  container.addEventListener('keydown', (e) => {
    if (e.key !== 'Tab') return;
    if (e.shiftKey) {
      if (document.activeElement === first) { e.preventDefault(); last.focus(); }
    } else {
      if (document.activeElement === last) { e.preventDefault(); first.focus(); }
    }
  });

  first.focus();
}
```

### Roving `tabindex` (Composite Widgets)
For widgets with multiple items (radio groups, toolbars, tab lists), only **one item** is in the tab sequence at a time. Arrow keys move within the widget.

```html
<div role="tablist">
  <button role="tab" tabindex="0"  aria-selected="true">Tab 1</button>
  <button role="tab" tabindex="-1" aria-selected="false">Tab 2</button>
  <button role="tab" tabindex="-1" aria-selected="false">Tab 3</button>
</div>
```

```ts
// On ArrowRight: move tabindex="0" to next tab, focus it
tabs.forEach((tab, i) => {
  tab.addEventListener('keydown', (e) => {
    if (e.key === 'ArrowRight') {
      tab.setAttribute('tabindex', '-1');
      const next = tabs[(i + 1) % tabs.length];
      next.setAttribute('tabindex', '0');
      next.focus();
    }
  });
});
```

---

## Keyboard Navigation Patterns

| Widget | Enter/Space | Arrow Keys | Escape | Home/End |
|--------|-------------|-----------|--------|----------|
| Button | Activate | — | — | — |
| Link | Follow | — | — | — |
| Dialog | — | — | Close | — |
| Tab list | Select tab | Navigate tabs | — | First/last tab |
| Menu | Select item | Navigate items | Close | First/last item |
| Combobox | Select/confirm | Navigate options | Close | First/last option |
| Tree | Expand/collapse | Navigate + expand | Collapse | First/last node |

Follow the **ARIA Authoring Practices Guide (APG)** at [w3.org/WAI/ARIA/apg](https://www.w3.org/WAI/ARIA/apg/) for authoritative interaction patterns.

---

## Form Accessibility

```html
<!-- ✅ Every input needs a visible, associated label -->
<label for="email">Email address</label>
<input type="email" id="email" aria-describedby="email-hint" autocomplete="email" />
<p id="email-hint">We'll never share your email.</p>

<!-- ✅ Error association -->
<input type="email" id="email" aria-invalid="true" aria-describedby="email-error" />
<p id="email-error" role="alert">Please enter a valid email address.</p>

<!-- ✅ Fieldset groups related controls -->
<fieldset>
  <legend>Notification preferences</legend>
  <label><input type="checkbox" name="email-notif"> Email</label>
  <label><input type="checkbox" name="sms-notif"> SMS</label>
</fieldset>
```

**Checklist**:
- Every input has a programmatically associated `<label>` (not just placeholder text)
- `autocomplete` attributes on personal data fields (SC 1.3.5)
- `aria-required="true"` or native `required` attribute on mandatory fields
- Errors are announced via `role="alert"` or `aria-live`; identified with `aria-invalid="true"`
- Error messages explain *what went wrong* and *how to fix it*, not just that an error occurred

---

## Color Contrast — WCAG vs. APCA

### WCAG 2.x Contrast Ratio (current standard)
Uses a **luminance ratio** formula. Minimum thresholds:

| Element | AA | AAA |
|---------|-----|-----|
| Normal text (<18pt) | 4.5:1 | 7:1 |
| Large text (≥18pt / 14pt bold) | 3:1 | 4.5:1 |
| UI components & graphics | 3:1 | — |

### APCA (Advanced Perceptual Contrast Algorithm)
APCA (targeted for WCAG 3.0) uses **lightness contrast (Lc)** and accounts for font size, weight, and spatial frequency — giving more accurate perceptual results than the simple ratio.

| Scenario | WCAG 2.x | APCA Lc |
|----------|----------|---------|
| Body text, 16px regular | 4.5:1 | Lc ≥ 75 |
| Large heading, 24px bold | 3:1 | Lc ≥ 60 |
| Placeholder / secondary text | 4.5:1 | Lc ≥ 30 |

**Trade-off**: WCAG 2.x is the legal standard today. APCA is more accurate but not yet normative. Run both — APCA catches cases where WCAG passes but text is still hard to read (e.g., thin fonts at moderate contrast).

**Tools**: [Colour Contrast Analyser](https://www.tpgi.com/color-contrast-checker/), [APCA Contrast Calculator](https://www.myndex.com/APCA/).

---

## Reduced Motion

Users with vestibular disorders can opt out of animation via the `prefers-reduced-motion` media query.

```css
/* Default: animations enabled */
.card {
  transition: transform 0.3s ease, opacity 0.3s ease;
}

/* Respect user preference — remove or dampen motion */
@media (prefers-reduced-motion: reduce) {
  .card {
    transition: opacity 0.1s ease; /* fade only; no transform */
  }

  /* Or kill all transitions globally */
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

Also check `prefers-reduced-motion` in JavaScript for programmatic animations (e.g., GSAP, Framer Motion):

```ts
const prefersReduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
if (!prefersReduced) {
  gsap.to(el, { y: -20, duration: 0.4 });
}
```

---

## Screen Reader Testing

### VoiceOver (macOS / iOS)
- **Activate**: Cmd + F5 (macOS) or triple-click Side Button (iOS)
- **Navigate**: VO+Arrow keys (VO = Ctrl+Option); Tab for focusable elements
- **Key checks**: Heading structure (VO+Cmd+H), landmark navigation (VO+Cmd+L), forms mode

### NVDA (Windows — free)
- **Download**: nvaccess.org (pairs with Firefox or Chrome)
- **Navigate**: Tab (focusable), H (headings), F (form fields), 1-6 (heading levels)
- **Browse vs. Forms mode**: NVDA switches automatically; Enter toggle with NVDA+Space

### Recommended Test Matrix

| Browser + AT | Platform | Priority |
|-------------|----------|---------|
| Safari + VoiceOver | macOS | High (largest AT share on Mac) |
| Chrome + NVDA | Windows | High (most common AT overall) |
| Chrome + JAWS | Windows | High (enterprise/gov users) |
| Safari + VoiceOver | iOS | High (dominant mobile AT) |
| Firefox + NVDA | Windows | Medium |

**Test script**: Navigate by headings → landmarks → forms → interactive widgets. Trigger live regions. Open and close modals. Activate all interactive elements by keyboard only.

---

## Automated Testing Tools

Automated tools catch ~30–40% of accessibility issues — manual and AT testing are still essential.

| Tool | Type | Best For |
|------|------|---------|
| **axe-core** | JS library / browser ext | CI integration, unit/e2e tests |
| **Lighthouse** | Chrome DevTools / CLI | Quick page audits, CI reports |
| **pa11y** | CLI / CI | Scripted multi-page scans |
| **Storybook a11y addon** | Dev environment | Per-component audits |
| **eslint-plugin-jsx-a11y** | Linter | Catch issues at author time |
| **Playwright / axe-playwright** | E2E | Full interaction flow audits |

```ts
// axe-core with Playwright — add to CI
import { checkA11y } from 'axe-playwright';

test('homepage has no critical a11y violations', async ({ page }) => {
  await page.goto('/');
  await checkA11y(page, undefined, {
    detailedReport: true,
    runOnly: { type: 'tag', values: ['wcag2aa', 'wcag22aa'] },
  });
});
```

```bash
# pa11y: scan a URL and fail CI if violations found
pa11y --standard WCAG2AA --reporter cli https://yoursite.com

# Lighthouse CLI: accessibility audit
lighthouse https://yoursite.com --only-categories=accessibility --output=json
```

---

## Trade-Off Summary

| Approach | Effort | Coverage | Notes |
|----------|--------|---------|-------|
| Semantic HTML only | Low | ~50% | Best ROI; prevents the majority of critical issues |
| Semantic HTML + ARIA | Medium | ~70% | Required for SPAs and custom widgets |
| + Focus management | Medium | ~80% | Essential for dialogs, menus, complex layouts |
| + Manual AT testing | High | ~95% | Catches what automated tools miss |
| + APCA contrast | Low add-on | Contrast edge cases | Use alongside WCAG 2.x today |

**The compound rule**: Start with semantic HTML (you get keyboard, naming, and roles for free), add ARIA only where native fails, implement proper focus management for all interactive overlays, then layer automated testing into CI. Manual AT testing with a screen reader is irreplaceable — budget at least one test session per major feature release.
