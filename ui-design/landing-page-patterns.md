---
title: "Landing Page Patterns"
category: ui-design
summary: "A practical guide to high-converting landing pages — anatomy, hero patterns, above-the-fold strategy, social proof, CTA design, pricing tables, FAQ sections, trust signals, page speed, A/B testing, mobile pages, copywriting frameworks (PAS, AIDA, BAB), and visual hierarchy."
sources:
  - web-research
updated: 2026-04-08T18:27:00.000Z
---

# Landing Page Patterns

> A practical guide to high-converting landing pages — anatomy, hero patterns, above-the-fold strategy, social proof, CTA design, pricing tables, FAQ sections, trust signals, page speed, A/B testing, mobile pages, copywriting frameworks (PAS, AIDA, BAB), and visual hierarchy.

---

## Anatomy of a High-Converting Landing Page

A landing page has one job: convert a visitor into a lead, customer, or subscriber. Every section either **builds trust**, **communicates value**, or **reduces friction** toward that single goal.

```
┌──────────────────────────────┐
│  Nav (logo + single CTA)     │  ← minimal nav, no escape links
├──────────────────────────────┤
│  Hero Section                │  ← above the fold, 3-second hook
│  (headline · subhead · CTA)  │
├──────────────────────────────┤
│  Social Proof Bar            │  ← logos or stat strip
├──────────────────────────────┤
│  Problem / Value Prop        │  ← "why this matters"
├──────────────────────────────┤
│  Features / Benefits         │  ← 3-column or alternating layout
├──────────────────────────────┤
│  Testimonials / Case Study   │  ← trust reinforcement
├──────────────────────────────┤
│  Pricing Table               │  ← anchored middle tier
├──────────────────────────────┤
│  FAQ                         │  ← objection handling
├──────────────────────────────┤
│  Final CTA / Footer          │  ← last chance conversion
└──────────────────────────────┘
```

**Key constraint:** Remove every element that doesn't serve conversion. No blog links, no full site nav, no social media icons that funnel traffic away.

---

## Hero Section Patterns

The hero is the most-tested real estate on any landing page. Three dominant patterns:

| Pattern | Best For | Risk |
|---------|----------|------|
| **Headline + CTA** | SaaS, services, lead gen | Weak copy kills it instantly |
| **Video Hero** | Complex products, demos, storytelling | Autoplay kills mobile perf; needs fallback |
| **Interactive Hero** | Tools, calculators, configurators | High dev cost; needs clear affordance |

### Headline + CTA (Most Common)

The proven workhorse. Structure: **outcome headline → clarifying subhead → primary CTA → trust micro-copy**.

```html
<section class="hero">
  <h1>Ship Features Twice as Fast</h1>
  <p class="subhead">The CI/CD platform built for growing engineering teams — zero config, instant deploys.</p>
  <a class="btn-primary" href="/signup">Start Free Trial</a>
  <span class="micro-trust">No credit card required · 5-minute setup</span>
</section>
```

**Headline formulas:**
- **Outcome-led:** "Get [Result] Without [Pain]"
- **Question:** "Tired of [Problem]?" → leads into PAS copy
- **Number:** "10,000 Teams Already Ship Daily" → social proof embedded

### Video Hero

Auto-playing muted background video conveys energy fast. Static poster image **must** load before video; lazy-load the video itself.

```html
<video autoplay muted loop playsinline poster="/hero-poster.jpg">
  <source src="/hero.webm" type="video/webm">
  <source src="/hero.mp4"  type="video/mp4">
</video>
```

**Trade-offs:**

| | Pro | Con |
|-|-----|-----|
| Video hero | High engagement, emotional pull | +2-4 s LCP on slow connections |
| Static hero | Fast load, accessible | Lower emotional impact |

### Interactive Hero

Calculators ("see your ROI"), product configurators, or live demos embedded above the fold. High intent — users who interact convert at 3-5× the rate of passive readers. Gate the result behind email capture for lead gen.

---

## Above-the-Fold Strategy

**Rule:** A visitor must understand *what you do*, *who it's for*, and *what to do next* — all without scrolling.

Checklist:
- [ ] Headline answers "what is this?"
- [ ] Subhead adds specificity (audience, mechanism, timeframe)
- [ ] Primary CTA is visible without scrolling on 1280px and 375px viewports
- [ ] No competing CTAs (one action only)
- [ ] Hero image/visual reinforces the headline (product screenshot > stock photo)

**Viewport test:** Open DevTools → toggle device toolbar → check 375×667 (iPhone SE) and 1440×900 (laptop). Both must show the CTA without scrolling.

---

## Social Proof Patterns

| Pattern | Placement | Impact |
|---------|-----------|--------|
| **Logo bar** ("trusted by") | Immediately below hero | Credibility anchoring |
| **Testimonial cards** | After features section | Objection neutralising |
| **Stats strip** ("10M users · 4.9★ · 99.9% uptime") | Hero or pricing | Quantified trust |
| **Case study pull-quote** | Near pricing | Purchase justification |
| **Review count badge** (G2, Capterra) | Near CTA | Third-party validation |

```html
<!-- Logo bar pattern -->
<div class="logo-bar" aria-label="Trusted by">
  <img src="/logos/stripe.svg" alt="Stripe" width="80" height="28">
  <img src="/logos/shopify.svg" alt="Shopify" width="90" height="28">
  <!-- ... -->
</div>
```

**Testimonial anatomy:** photo + full name + title + company + specific result ("reduced deploy time by 60%"). Vague praise ("great product!") adds no conversion value.

---

## CTA Design

### Button Copy

Generic copy loses. Outcome-specific copy wins.

| Weak | Strong |
|------|--------|
| Submit | Get My Free Report |
| Sign Up | Start Building Free |
| Learn More | See How It Works |
| Buy Now | Get Instant Access |

**Formula:** Verb + Outcome/Object + (optional) Qualifier  
`Start your free trial` · `Download the checklist` · `Claim your 20% discount`

### Placement

- **Primary CTA:** hero, end of each major section, pricing, final footer
- **Sticky CTA bar:** appears after 50% scroll on mobile; keep it 44px+ tall
- **Inline CTAs:** within prose ("…which is why [10,000 teams use X →]")

### Urgency Patterns

Use genuine scarcity only — manufactured urgency destroys trust on repeat visits.

```html
<!-- Genuine deadline -->
<div class="urgency-banner">
  🔥 Founding price ends <time datetime="2026-04-15">April 15</time>
</div>

<!-- Stock scarcity -->
<span class="seats-left">Only 3 seats left at this price</span>
```

---

## Pricing Table Patterns

Three-tier pricing (Good / Better / Best) is the industry default. The middle tier should be **anchored visually** as "Most Popular".

```
┌──────────┬───────────────┬──────────┐
│  Starter │  Pro ★ Popular│ Enterprise│
│  $29/mo  │   $79/mo      │  Custom  │
│──────────│───────────────│──────────│
│ 5 users  │  Unlimited    │  SLA     │
│ 10 GB    │  100 GB       │  Custom  │
│ Email    │  + Phone      │  + CSM   │
│  [Start] │  [Start Free] │ [Contact]│
└──────────┴───────────────┴──────────┘
```

**Anchoring:** List the highest tier first (left-to-right in Western layouts) to make middle price feel reasonable.  
**Toggle:** Monthly/Annual billing toggle with "Save 20%" label increases annual plan uptake.  
**Feature rows:** Lead with benefits, not feature names. "Priority support" → "Replies in < 2 hours".

---

## FAQ Sections

FAQs are structured objection-handling. Write questions from the visitor's perspective, not marketing copy.

Common high-value FAQs:
- "How long does setup take?" (effort objection)
- "Can I cancel anytime?" (commitment risk)
- "Do you offer refunds?" (financial risk)
- "Is my data secure?" (privacy concern)
- "How is this different from [competitor]?" (alternative consideration)

Use `<details>`/`<summary>` for native accordion — zero JS required, accessible, and fast.

```html
<details>
  <summary>Can I cancel anytime?</summary>
  <p>Yes. Cancel with one click from your dashboard. No questions, no calls required.</p>
</details>
```

---

## Trust Signals

| Signal | Implementation |
|--------|---------------|
| SSL badge | HTTPS padlock is table stakes — also display security copy near forms |
| Money-back guarantee | Badge + policy link near CTA ("30-day no-risk guarantee") |
| Awards / press mentions | "As seen in Forbes, TechCrunch" with logo strip |
| Team photos / about link | Humanises brand; reduces fraud perception |
| Privacy reassurance | "We never sell your data" near email capture |
| Real-time activity | "243 people signed up today" (genuine data only) |

---

## Page Speed Impact on Conversion

Google's data: **every 100 ms of latency costs ~1% of conversions**. For e-commerce, a 1-second delay drops conversions by ~7%.

| Core Web Vital | Target | Conversion Impact |
|----------------|--------|-------------------|
| LCP (Largest Contentful Paint) | < 2.5 s | Directly tied to bounce rate |
| CLS (Cumulative Layout Shift) | < 0.1 | Layout jumps kill trust |
| INP (Interaction to Next Paint) | < 200 ms | Friction on CTA click |

**Quick wins:**
```html
<!-- Preload hero image -->
<link rel="preload" as="image" href="/hero.webp" fetchpriority="high">

<!-- Defer non-critical JS -->
<script src="/analytics.js" defer></script>

<!-- Use modern image formats -->
<picture>
  <source srcset="/hero.avif" type="image/avif">
  <source srcset="/hero.webp" type="image/webp">
  <img src="/hero.jpg" alt="..." width="1200" height="630">
</picture>
```

Inline critical CSS for above-the-fold content; load the rest asynchronously.

---

## A/B Testing Landing Pages

Test one variable at a time. Priority order by typical impact:

1. **Headline** — highest variance; swap value prop angle
2. **CTA copy** — verb/outcome/qualifier variants
3. **Hero image** — product screenshot vs. person vs. abstract
4. **Social proof type** — logos vs. stats vs. testimonials
5. **Pricing structure** — monthly default vs. annual default
6. **Form length** — email only vs. email + name vs. full form

**Minimum sample size:** ~1,000 conversions per variant for statistical significance at 95% confidence. Use a calculator (e.g., Evan Miller's) before calling a winner.

**Common traps:**
- Stopping early on a positive trend ("peeking problem")
- Running too many variants simultaneously (traffic dilution)
- Testing cosmetic changes before messaging

---

## Mobile Landing Pages

Mobile-first is mandatory — 60-70% of landing page traffic is mobile for most SaaS/e-commerce.

```css
/* Sticky CTA — mobile only */
@media (max-width: 768px) {
  .sticky-cta {
    position: fixed;
    bottom: 0;
    width: 100%;
    padding: 12px 16px;
    background: white;
    box-shadow: 0 -2px 8px rgba(0,0,0,0.1);
    z-index: 100;
  }
}
```

**Mobile-specific considerations:**
- Tap targets ≥ 44×44 px (Apple HIG) / 48×48 dp (Material)
- Single-column layout; avoid horizontal scroll
- Forms: use `inputmode="email"` / `type="tel"` to trigger correct keyboard
- Hero video: use `playsinline` to prevent fullscreen hijack on iOS
- Above-the-fold CTA must not be covered by browser chrome (use `dvh` units)

```css
.hero { min-height: 100dvh; } /* accounts for browser chrome */
```

---

## Copywriting Frameworks

### PAS — Problem · Agitate · Solution

Best for high-pain, high-awareness audiences. Lead with the wound.

```
Problem:  "You're losing leads because your forms take 30 seconds to load."
Agitate:  "Every slow second costs you a potential customer — and they're going to your competitor."
Solution: "Formly loads in under 500 ms, works offline, and auto-saves. Never lose a lead again."
```

### AIDA — Attention · Interest · Desire · Action

Classic sequential funnel — best for long-form sales pages.

```
Attention: "What if you could launch a landing page in 10 minutes?"
Interest:  "Most builders require design skills or a developer. We don't."
Desire:    "10,000 marketers already ship pages with zero code — and convert 37% better."
Action:    "Start your free trial — no credit card required."
```

### BAB — Before · After · Bridge

Ideal for transformation products (fitness, productivity, finance).

```
Before: "You're spending 8 hours a week on manual reporting."
After:  "Imagine having that time back — with reports that write themselves."
Bridge: "Dashify connects to 50+ data sources and builds your board report automatically."
```

**Rule:** Match the framework to awareness level. Cold traffic → PAS or BAB (lead with pain). Warm/retargeted traffic → AIDA (lead with outcome).

---

## Visual Hierarchy for Conversion

Visual hierarchy guides the eye toward the CTA in a predictable sequence.

| Level | Element | Technique |
|-------|---------|-----------|
| 1st | Headline | Largest type (48-72px), high contrast |
| 2nd | Hero image / video | Full-bleed or dominant size |
| 3rd | Subhead | 18-24px, muted colour |
| 4th | CTA button | High-contrast fill, generous padding |
| 5th | Trust micro-copy | Small, de-emphasised (grey) |

**F-pattern vs. Z-pattern:**
- **F-pattern** — text-heavy pages; users scan headline, then two horizontal strips
- **Z-pattern** — sparse pages; eye travels top-left → top-right → diagonal → bottom-right (natural CTA landing zone)

**Colour contrast:** CTA button should not share the brand's primary colour if that colour appears everywhere — use a high-contrast accent. Orange and green CTAs on blue/neutral backgrounds consistently outperform brand-coloured buttons in tests.

**Whitespace as hierarchy:** Surround the CTA with generous whitespace (≥ 32px on all sides). Clutter around the button is the #1 silent conversion killer.

---

## Quick-Reference Trade-offs

| Decision | Option A | Option B | Pick when |
|----------|----------|----------|-----------|
| Nav on landing page | Full nav | Logo only | Full nav if brand is unknown; logo-only for paid traffic |
| Form placement | Above fold | Below fold | Above fold for single-field (email); below fold for multi-step |
| Video autoplay | Yes (muted) | No | Yes if LCP stays < 2.5 s; otherwise poster + play button |
| Pricing on page | Visible | "Contact us" | Visible for self-serve; hidden only for true enterprise |
| Single vs multi-step form | Single | Multi-step | Multi-step increases completion by 10-30% for > 3 fields |

---

*Related: [[mobile-first-and-responsive-design]] · [[typography-for-ui]] · [[color-theory-and-palette-design]]*
