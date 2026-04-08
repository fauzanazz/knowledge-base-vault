---
title: "Dark Mode Implementation"
category: ui-design
summary: "A comprehensive guide to dark mode beyond simple color inversion — covering elevation through lightness, Material Design guidelines, semantic color tokens, CSS implementation strategies, palette design, image handling, shadow behavior, and system preference detection with manual override."
sources:
  - web-research
updated: 2026-04-08T18:30:00.000Z
---

# Dark Mode Implementation

> Dark mode is not a CSS filter — it is a parallel visual language. Done well, it requires a considered color system, elevation semantics, component-specific adaptations, and persistent user preference management. This article covers the full stack.

---

## Why Dark Mode Is Not Color Inversion

A naive dark mode slaps `filter: invert(1)` on the `<body>` and calls it done. The result is jarring: photos look like negatives, brand colors flip to unrecognizable complements, and text loses its intended hierarchy. Real dark mode is a **separate theme** with its own semantic intent.

The core differences:

| Dimension | Light Mode | Dark Mode |
|---|---|---|
| Background base | Near-white (`#F8F9FA`) | Dark gray (`#121212`), not pure black |
| Elevation signal | Shadows (darker) | Lightness increase (lighter surfaces) |
| Color saturation | Full saturation | Slightly desaturated to reduce eye strain |
| Text contrast | Dark on light | Light on dark — different contrast math |
| Shadows | High visibility | Nearly invisible — replaced by surface lightness |
| Images | Default | May need brightness/contrast adjustment |

---

## Elevation Through Lightness (Material Design Dark Theme)

In light mode, **elevation is communicated with shadows** — higher cards cast deeper shadows. In dark mode, shadows nearly disappear against dark backgrounds. Material Design 3 solves this by encoding elevation as **surface lightness**: higher surfaces get a slightly lighter background via a white overlay at increasing opacity.

**Material Design dark surface overlay levels:**

| Elevation Level | Overlay Opacity | Approximate Surface Color |
|---|---|---|
| 0dp (base) | 0% | `#121212` |
| 1dp | 5% | `#1E1E1E` |
| 2dp | 7% | `#222222` |
| 4dp | 9% | `#252525` |
| 8dp | 11% | `#272727` |
| 16dp | 13% | `#2C2C2C` |
| 24dp | 15% | `#2E2E2E` |

This creates a **surface hierarchy** — dialogs feel above cards, which feel above the page — without relying on shadows. Implement this with CSS custom properties:

```css
/* Base dark surface */
--surface-0: #121212;

/* Elevated surfaces — white overlay via mix-blend or explicit values */
--surface-1: color-mix(in srgb, white 5%, #121212);
--surface-2: color-mix(in srgb, white 7%, #121212);
--surface-4: color-mix(in srgb, white 9%, #121212);
--surface-8: color-mix(in srgb, white 11%, #121212);
```

> **Key insight:** Never use pure black (`#000000`) as your dark background. Pure black creates excessive contrast with any content, strains the eye on OLED, and eliminates any room for elevation differentiation. `#121212` is the Material Design standard; `#0F0F0F`–`#1A1A1A` is the practical range.

---

## Semantic Color Tokens for Dark Mode

The right architecture is a **two-tier token system**: primitive tokens (raw color values) are aliased by semantic tokens whose *names* describe purpose, not value. Dark mode is then a semantic token remap, not a new component stylesheet.

```json
// tokens.json (DTCG format)

// Primitives — the full palette
"color": {
  "gray": {
    "50":  { "$value": "#F9FAFB" },
    "900": { "$value": "#111827" },
    "950": { "$value": "#0A0F1A" }
  },
  "blue": {
    "400": { "$value": "#60A5FA" },
    "600": { "$value": "#2563EB" }
  }
},

// Semantic tokens — aliased by theme
"surface": {
  "background": { "$value": "{color.gray.50}" },    // light default
  "card":        { "$value": "#FFFFFF" },
  "overlay":     { "$value": "rgba(0,0,0,0.5)" }
},
"text": {
  "primary":     { "$value": "{color.gray.900}" },
  "secondary":   { "$value": "{color.gray.600}" }
},
"interactive": {
  "primary":     { "$value": "{color.blue.600}" }
}
```

In a separate dark theme file, only semantic tokens are overridden — primitives stay the same:

```json
// tokens.dark.json
"surface": {
  "background": { "$value": "#121212" },
  "card":        { "$value": "{color.gray.900}" },
  "overlay":     { "$value": "rgba(0,0,0,0.7)" }
},
"text": {
  "primary":     { "$value": "#E8EAED" },
  "secondary":   { "$value": "#9AA0A6" }
},
"interactive": {
  "primary":     { "$value": "{color.blue.400}" }  // lighter for dark bg contrast
}
```

> **Note the interactive token shift:** In dark mode, blue `400` (lighter) replaces blue `600` (darker). Dark backgrounds require *lighter* variants of brand colors to maintain WCAG AA contrast (4.5:1 for normal text).

---

## CSS Implementation Strategies

Three approaches exist, each with trade-offs:

### 1. `prefers-color-scheme` Media Query

Automatically respects the OS-level preference. No JavaScript required.

```css
:root {
  --bg: #ffffff;
  --text: #111827;
  --surface: #f3f4f6;
}

@media (prefers-color-scheme: dark) {
  :root {
    --bg: #121212;
    --text: #e8eaed;
    --surface: #1e1e1e;
  }
}
```

**Trade-off:** No manual override. Users cannot toggle dark mode independent of OS.

---

### 2. Class Toggle

A `.dark` class on `<html>` or `<body>` is toggled via JavaScript. Frameworks like Tailwind CSS use `dark:` utilities with this approach.

```css
:root {
  --bg: #ffffff;
  --text: #111827;
}

.dark {
  --bg: #121212;
  --text: #e8eaed;
}
```

```js
// Toggle
document.documentElement.classList.toggle('dark');
```

**Trade-off:** Requires JS; a flash of unstyled content (FOUC) can appear if the class is applied after initial paint. Mitigate by inlining a synchronous script in `<head>` that reads `localStorage` and applies the class before render.

---

### 3. `data-theme` Attribute

A clean alternative to class toggling. Supports multiple themes beyond just light/dark.

```css
[data-theme="light"] { --bg: #ffffff; --text: #111827; }
[data-theme="dark"]  { --bg: #121212; --text: #e8eaed; }
[data-theme="high-contrast"] { --bg: #000000; --text: #ffffff; }
```

```js
document.documentElement.setAttribute('data-theme', 'dark');
```

**Trade-off:** Slightly more verbose CSS; excellent for multi-theme products.

---

### Combining System Preference + Manual Override + Persistence

```js
// theme.js — inline in <head> to prevent FOUC
(function () {
  const stored = localStorage.getItem('theme');
  const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
  const theme = stored ?? (prefersDark ? 'dark' : 'light');
  document.documentElement.setAttribute('data-theme', theme);
})();

// Public API
function setTheme(theme) {
  document.documentElement.setAttribute('data-theme', theme);
  localStorage.setItem('theme', theme);
}

// Listen for OS changes if no manual override
window.matchMedia('(prefers-color-scheme: dark)')
  .addEventListener('change', (e) => {
    if (!localStorage.getItem('theme')) {
      setTheme(e.matches ? 'dark' : 'light');
    }
  });
```

This gives **three behaviors**: use stored preference if set → fall back to OS preference → update automatically when OS changes (only if user hasn't manually overridden).

---

## Dark Mode Palette Design

### Desaturation

Highly saturated colors on dark backgrounds vibrate and cause eye strain (the Helmholtz–Kohlrausch effect). **Reduce chroma by 10–20%** for brand and accent colors in dark mode.

```css
/* Light mode: vivid blue */
--interactive-primary: hsl(221, 83%, 53%);

/* Dark mode: same hue, lower saturation + higher lightness */
--interactive-primary: hsl(221, 65%, 65%);
```

### Dark Gray Scale

Build a perceptually consistent gray ramp rather than arbitrary values:

| Token | Value | Use |
|---|---|---|
| `gray-950` | `#0A0F1A` | Deepest — rarely used alone |
| `gray-900` | `#121212` | Page background |
| `gray-850` | `#1A1A2E` | Slightly elevated surface |
| `gray-800` | `#1E1E1E` | Cards at 1dp elevation |
| `gray-750` | `#252525` | Cards at 4dp elevation |
| `gray-700` | `#2C2C2C` | Modals / drawers |
| `gray-600` | `#3C3C3C` | Input borders, dividers |
| `gray-400` | `#9AA0A6` | Placeholder text, secondary labels |
| `gray-100` | `#E8EAED` | Primary text |

---

## Image Handling in Dark Mode

Images generally do not need inversion, but some adjustments improve comfort.

**Raster photos:** Reduce brightness slightly and increase contrast to compensate for the dark surround effect.

```css
@media (prefers-color-scheme: dark) {
  img:not([data-no-dim]) {
    filter: brightness(0.9) contrast(1.05);
  }
}
```

**Icons and SVG illustrations:** Prefer SVGs that inherit `currentColor` so they adapt automatically. For complex multi-color illustrations, provide separate dark-mode variants using `<picture>`:

```html
<picture>
  <source srcset="/images/hero-dark.webp" media="(prefers-color-scheme: dark)">
  <img src="/images/hero-light.webp" alt="Hero illustration">
</picture>
```

**Logos on dark backgrounds:** Light-mode logos often become invisible. Always provide a white/light variant and serve it conditionally.

---

## Shadow Behavior in Dark Mode

Shadows lose effectiveness on dark backgrounds — a `box-shadow: 0 4px 12px rgba(0,0,0,0.3)` is nearly invisible against `#121212`. The fix is **dual-pronged**:

1. **Use surface lightness for elevation** (as covered above).
2. **Replace or supplement shadows with inner glow / border highlights.**

```css
/* Light mode card */
.card {
  background: #ffffff;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

/* Dark mode card — elevation via lightness + subtle border */
[data-theme="dark"] .card {
  background: var(--surface-2);            /* slightly lighter than bg */
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.6); /* much stronger alpha needed */
  border: 1px solid rgba(255, 255, 255, 0.06);
}
```

---

## Text Contrast on Dark Backgrounds

WCAG 2.1 requires 4.5:1 contrast for normal text and 3:1 for large text (18pt+/14pt+ bold) against backgrounds. On dark surfaces, use a **luminance hierarchy** rather than pure white for all text:

| Role | Value | Contrast on `#121212` |
|---|---|---|
| Primary text | `#E8EAED` | ~13:1 — High |
| Secondary text | `#9AA0A6` | ~5.3:1 — AA Pass |
| Disabled text | `#5F6368` | ~2.8:1 — Intentionally below threshold |
| Link / interactive | `#60A5FA` (blue-400) | ~5.2:1 — AA Pass |

Avoid pure white (`#FFFFFF`) for body text — the extreme contrast (~21:1) causes halation on OLED and increases legibility fatigue.

---

## Component-Specific Considerations

### Cards
- Use `--surface-1` or `--surface-2`, not the page background, to maintain visual separation.
- Add a hairline border (`rgba(255,255,255,0.06)`) when card bg is close to page bg.

### Modals / Dialogs
- Use `--surface-4` or higher elevation to float clearly above content.
- Overlay/backdrop should darken further: `rgba(0,0,0,0.7)` instead of `0.5`.

### Inputs / Form Controls
- Default browser inputs inherit poorly; always override background and border.
- Use a slightly lighter surface than the page (`--surface-1`) for the input field.
- Focus ring must be visible: a blue or accent ring at 2px offset works against dark backgrounds.

```css
[data-theme="dark"] input,
[data-theme="dark"] textarea {
  background: var(--surface-1);        /* #1e1e1e */
  border: 1px solid var(--gray-600);   /* #3c3c3c */
  color: var(--text-primary);
}

[data-theme="dark"] input:focus {
  outline: 2px solid var(--interactive-primary);
  outline-offset: 2px;
}
```

### Navigation / Sidebars
- Active states: use a low-opacity accent background (`rgba(96,165,250,0.12)`) rather than a solid fill.
- Hover states: `rgba(255,255,255,0.06)` provides a subtle, non-distracting lift.

---

## Testing Dark Mode

| Method | Tool | What to Check |
|---|---|---|
| OS toggle | macOS/Windows system settings | Full system preference flow |
| DevTools emulation | Chrome DevTools → Rendering → Emulate CSS media feature | Quick visual iteration |
| Forced dark mode | Chrome `--force-dark-mode` flag | Stress-test for unhandled colors |
| Contrast audit | axe DevTools, Stark, Colour Contrast Analyser | WCAG ratios for all text/bg combos |
| Screenshot diffing | Playwright `colorScheme: 'dark'`, Percy | Regression detection in CI |
| Manual override persistence | Clear localStorage, check fallback; set preference, reload, verify | Preference round-trip |

```js
// Playwright dark mode test
test('renders dark mode correctly', async ({ page }) => {
  await page.emulateMedia({ colorScheme: 'dark' });
  await page.goto('/');
  await expect(page.locator('body')).toHaveCSS(
    'background-color', 'rgb(18, 18, 18)'
  );
  await expect(page).toHaveScreenshot('home-dark.png');
});
```

---

## Summary: Dark Mode Decision Checklist

- [ ] Use dark gray (`#121212`) not pure black as the base background
- [ ] Encode elevation via surface lightness, not shadows alone
- [ ] Define a two-tier token system: primitives + semantic aliases per theme
- [ ] Choose a CSS strategy: media query only, class toggle, or `data-theme` attribute
- [ ] Persist manual override in `localStorage`; apply synchronously before paint to prevent FOUC
- [ ] Desaturate brand/accent colors in dark mode by 10–20%
- [ ] Provide dark-mode image variants or apply `brightness`/`contrast` filters
- [ ] Replace/supplement shadows with surface lightness + subtle borders
- [ ] Verify WCAG contrast for all text/background combinations
- [ ] Test card, modal, input, and nav components individually
- [ ] Run Playwright dark mode screenshot tests in CI
