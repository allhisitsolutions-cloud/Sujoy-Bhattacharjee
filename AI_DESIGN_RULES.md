# AI_DESIGN_RULES.md

**120 Army — AI Design Rules for Every Page**

This document is the authoritative rulebook for how every screen must be built. Read it in full before generating any UI. Every rule is non-negotiable unless explicitly overridden in a specific component's requirements.

Standard: every page must feel comparable to **Apple**, **Linear**, and **Stripe**.

---

## Table of Contents

1. [Core Design Philosophy](#1-core-design-philosophy)
2. [Page Structure Rules](#2-page-structure-rules)
3. [Background & Atmosphere](#3-background--atmosphere)
4. [Typography Rules](#4-typography-rules)
5. [Color Application Rules](#5-color-application-rules)
6. [Glassmorphism Rules](#6-glassmorphism-rules)
7. [Animation Rules](#7-animation-rules)
8. [Layout & Spacing Rules](#8-layout--spacing-rules)
9. [Component Rules](#9-component-rules)
10. [Loading & Empty States](#10-loading--empty-states)
11. [Mobile Rules](#11-mobile-rules)
12. [Anti-Patterns — Never Do These](#12-anti-patterns--never-do-these)
13. [Page-Type Patterns](#13-page-type-patterns)
14. [Quick-Reference Checklist](#14-quick-reference-checklist)

---

## 1. Core Design Philosophy

### The Three Tests

Before generating any page, apply these tests mentally:

**Test 1 — The Apple Test**
Would this feel at home on apple.com? Is every element intentional, refined, and purposeful? Is there generous whitespace? Does it feel like it was designed by people who care deeply?

**Test 2 — The Linear Test**
Does it feel technically sophisticated? Is the dark mode executed with depth and nuance rather than just a black background? Does every interaction feel smooth and considered?

**Test 3 — The Stripe Test**
Is the hierarchy crystal clear? Can a new user understand the purpose of this page in under 3 seconds? Is every word earning its place?

If any test fails, redesign before writing code.

### The Spiritual Dimension

120 Army is not a generic SaaS product. It is a sacred digital space. This means:

- Imagery, copy, and layout should evoke reverence, warmth, and hope — not corporate productivity
- Gold (`#D4AF37`) is sacred — use it for moments that matter, not decoration
- Empty space is intentional — it creates room to breathe, reflect, and pray
- Animations should feel like a gentle invitation, not a performance
- The platform should feel like entering a prayer room, not a dashboard

---

## 2. Page Structure Rules

### Every Page Has Exactly These Layers

```
Layer 1 — Background atmosphere    (fixed, behind everything)
Layer 2 — Content surfaces         (glass cards, panels)
Layer 3 — Interactive elements     (buttons, inputs, controls)
Layer 4 — Overlays                 (modals, drawers, tooltips)
Layer 5 — Notifications/toasts     (top of z-stack)
```

Never collapse these layers. Never place content directly on a flat solid background.

### Page Anatomy

Every page must have a defined **above-the-fold moment** — the first thing the user sees must communicate purpose, create emotion, and invite action. This is not negotiable.

```
┌─────────────────────────────────────────┐
│  NAVIGATION                             │  ← fixed, glass
├─────────────────────────────────────────┤
│                                         │
│  HERO / ABOVE-FOLD                      │  ← full emotional impact
│  (gradient bg, headline, subtext, CTA)  │
│                                         │
├─────────────────────────────────────────┤
│                                         │
│  SECTION 1                              │  ← unique layout, not a card grid
│                                         │
├─────────────────────────────────────────┤
│  SECTION 2 ...                          │  ← different rhythm from Section 1
└─────────────────────────────────────────┘
```

### Section Variety Rule

No two consecutive sections may use the same layout pattern. Alternate between:
- Full-width immersive
- Asymmetric left/right split
- Centered single-column
- Feature grid (non-uniform)
- Horizontal scroll
- Bento-box layout

Repetitive section patterns signal a generic template. Break the rhythm.

---

## 3. Background & Atmosphere

### Rule: Never Use a Flat Solid Background

The base background `#0F172A` must always be layered with atmosphere. Every page needs at least two of:

```tsx
{/* Deep radial glow — position varies per page */}
<div className="fixed inset-0 -z-10 overflow-hidden pointer-events-none">

  {/* Primary atmospheric glow */}
  <div className="absolute -top-40 left-1/2 -translate-x-1/2 w-[800px] h-[600px]
                  bg-[#D4AF37]/8 rounded-full blur-[120px]" />

  {/* Secondary glow — offset for depth */}
  <div className="absolute top-1/2 -right-40 w-[600px] h-[600px]
                  bg-[#3B82F6]/6 rounded-full blur-[100px]" />

  {/* Bottom anchor glow */}
  <div className="absolute -bottom-20 left-1/4 w-[500px] h-[400px]
                  bg-[#D4AF37]/5 rounded-full blur-[80px]" />

</div>
```

### Noise Texture

Add subtle grain to every page for depth and tactility:

```tsx
<div className="fixed inset-0 -z-10 opacity-[0.03] pointer-events-none"
     style={{ backgroundImage: "url('/noise.png')" }} />
```

### Grid / Mesh Overlays (optional, for hero sections)

```tsx
{/* Subtle dot grid */}
<div className="absolute inset-0 opacity-[0.04]"
     style={{
       backgroundImage: 'radial-gradient(circle, #D4AF37 1px, transparent 1px)',
       backgroundSize: '32px 32px'
     }} />
```

### Light Beam (hero sections only)

```tsx
{/* Crepuscular beam — use sparingly */}
<div className="absolute top-0 left-1/2 -translate-x-1/2
                w-px h-64 bg-gradient-to-b from-[#D4AF37]/60 to-transparent" />
```

---

## 4. Typography Rules

### Hierarchy Is Sacred

Never guess at font sizes. Always use the design system scale. A page must have exactly one `display` element above the fold — it commands attention.

```tsx
{/* ✅ Correct — clear hierarchy */}
<h1 className="font-display text-5xl md:text-7xl font-bold leading-[1.1] tracking-tight">
  United in Prayer
</h1>
<p className="text-lg text-[#94A3B8] leading-relaxed max-w-xl">
  Join 120,000 believers praying together across 180 nations.
</p>
```

```tsx
{/* ❌ Wrong — same weight and size everywhere */}
<h1 className="text-2xl font-bold">United in Prayer</h1>
<p className="text-2xl">Join 120,000 believers...</p>
```

### Gold Gradient on Hero Text

Apply the gold gradient exclusively to the most emotionally significant word or phrase in a hero headline — not to entire paragraphs.

```tsx
<h1 className="font-display text-6xl font-bold">
  The World Is{' '}
  <span className="bg-gradient-to-r from-[#D4AF37] via-[#E8CC6A] to-[#D4AF37]
                   bg-clip-text text-transparent">
    Praying
  </span>
</h1>
```

### Readable Contrast

- Body text on dark backgrounds: minimum `#94A3B8` (never `#64748B` or darker for body)
- Headings: `#F8FAFC`
- Labels/captions: `#94A3B8`
- Disabled: `#475569`

### Line Length

Constrain paragraph text to `max-w-prose` (65ch) or `max-w-xl`. Never let a paragraph span the full page width — it destroys readability.

---

## 5. Color Application Rules

### Gold Is Sacred — Use It Intentionally

Gold (`#D4AF37`) should appear on a page no more than **3–5 times**. Its power comes from scarcity.

**Correct gold usage:**
- Primary CTA button
- Active nav state
- Key stat / number highlight
- Hero text gradient accent
- Card border on featured/promoted content

**Incorrect gold usage:**
- Decorating every section heading
- Background fill on large areas
- Icon color on every icon
- Generic dividers

### Blue Is Supporting

Accent blue (`#3B82F6`) is for information density — links, secondary badges, progress indicators, informational callouts. It should never compete with gold on the same element.

### The Hierarchy of Fills

```
Most prominent  →  Gold gradient
Supporting      →  Blue
Neutral action  →  White/8 glass
Inactive        →  White/4
```

---

## 6. Glassmorphism Rules

Glass is the primary surface material. Every card, panel, modal, and floating element uses glass.

### Standard Glass Card — Copy This Pattern

```tsx
<div className="relative overflow-hidden rounded-2xl
                bg-gradient-to-br from-white/8 to-white/2
                backdrop-blur-xl border border-white/8
                shadow-[0_8px_32px_rgba(0,0,0,0.4)]
                hover:border-white/14 hover:shadow-[0_12px_48px_rgba(0,0,0,0.5)]
                hover:-translate-y-1 transition-all duration-300">

  {/* Decorative inner glow — always include on feature cards */}
  <div className="absolute -top-16 -right-16 w-48 h-48
                  bg-[#D4AF37]/8 rounded-full blur-3xl pointer-events-none" />

  {/* Content */}
  <div className="relative z-10 p-6">
    {children}
  </div>

</div>
```

### Glass Depth Stack — Use When Layering Panels

| Layer | Background | Blur | Border |
|---|---|---|---|
| Page surface | `from-white/8 to-white/2` | `blur-xl` | `white/8` |
| Elevated panel | `from-white/10 to-white/4` | `blur-2xl` | `white/10` |
| Modal/overlay | `from-white/12 to-white/6` | `blur-3xl` | `white/12` |

Never stack more than 3 layers. Never place glass on glass on glass — one surface per depth level.

### Gold Glass — Featured Content Only

```tsx
<div className="bg-gradient-to-br from-[#D4AF37]/12 to-[#D4AF37]/3
                backdrop-blur-xl border border-[#D4AF37]/20 rounded-2xl
                shadow-[0_8px_32px_rgba(0,0,0,0.4),0_0_48px_rgba(212,175,55,0.08)]">
```

Use for: featured prayer requests, subscription upsell cards, milestone announcements.

---

## 7. Animation Rules

### The Golden Rule of Animation

Animations serve the user's understanding and emotional experience. They are never decorative delay or attention-grabbing performance. Every animated element must arrive **before** the user needs it.

### Mandatory Animations

Every page must implement:

1. **Page entrance** — sections fade up as they enter the viewport
2. **Interactive hover states** — every clickable card lifts (`-translate-y-1`) and brightens
3. **CTA pulse** — primary button has a subtle gold glow pulse on idle
4. **Staggered lists** — list items enter with a stagger, never all at once

### Standard Page Entrance Pattern

```tsx
'use client';
import { motion } from 'framer-motion';

const fadeUp = {
  hidden:  { opacity: 0, y: 32 },
  visible: { opacity: 1, y: 0 },
};

const stagger = {
  visible: { transition: { staggerChildren: 0.1, delayChildren: 0.1 } },
};

// Section wrapper
<motion.section
  variants={stagger}
  initial="hidden"
  whileInView="visible"
  viewport={{ once: true, margin: '-80px' }}
>
  <motion.h2 variants={fadeUp} transition={{ duration: 0.6, ease: [0.0, 0.0, 0.2, 1] }}>
    Global Prayer
  </motion.h2>

  <motion.div variants={stagger} className="grid grid-cols-3 gap-6">
    {items.map((item) => (
      <motion.div key={item.id} variants={fadeUp}
                  transition={{ duration: 0.5, ease: [0.0, 0.0, 0.2, 1] }}>
        <Card {...item} />
      </motion.div>
    ))}
  </motion.div>
</motion.section>
```

### Interactive Micro-animations

```tsx
{/* Card hover — always use whileHover, never just CSS */}
<motion.div
  whileHover={{ y: -4, transition: { duration: 0.2 } }}
  whileTap={{ scale: 0.98 }}
>

{/* Button press */}
<motion.button
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.97 }}
>

{/* Gold CTA pulse */}
<motion.button
  animate={{ boxShadow: [
    '0 0 20px rgba(212,175,55,0.2)',
    '0 0 40px rgba(212,175,55,0.4)',
    '0 0 20px rgba(212,175,55,0.2)',
  ]}}
  transition={{ duration: 3, repeat: Infinity, ease: 'easeInOut' }}
>
```

### Number Counter Animation

For stats/metrics — never render a static number:

```tsx
import { useMotionValue, useSpring, useInView } from 'framer-motion';
import { useEffect, useRef } from 'react';

function AnimatedNumber({ value }: { value: number }) {
  const ref = useRef(null);
  const isInView = useInView(ref, { once: true });
  const motionVal = useMotionValue(0);
  const spring = useSpring(motionVal, { stiffness: 60, damping: 15 });

  useEffect(() => {
    if (isInView) motionVal.set(value);
  }, [isInView, value, motionVal]);

  return <motion.span ref={ref}>{spring}</motion.span>;
}
```

### Modal / Drawer Entrance

```tsx
<AnimatePresence>
  {isOpen && (
    <>
      {/* Backdrop */}
      <motion.div
        className="fixed inset-0 bg-black/60 backdrop-blur-sm z-50"
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
        transition={{ duration: 0.2 }}
        onClick={onClose}
      />
      {/* Panel */}
      <motion.div
        className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 z-50 ..."
        initial={{ opacity: 0, scale: 0.95, y: 8 }}
        animate={{ opacity: 1, scale: 1, y: 0 }}
        exit={{ opacity: 0, scale: 0.95, y: 8 }}
        transition={{ duration: 0.25, ease: [0.0, 0.0, 0.2, 1] }}
      >
        {children}
      </motion.div>
    </>
  )}
</AnimatePresence>
```

### Respecting User Preferences

```tsx
import { useReducedMotion } from 'framer-motion';

function AnimatedCard({ children }) {
  const reduce = useReducedMotion();
  return (
    <motion.div
      whileHover={reduce ? {} : { y: -4 }}
      transition={{ duration: reduce ? 0 : 0.2 }}
    >
      {children}
    </motion.div>
  );
}
```

---

## 8. Layout & Spacing Rules

### The Breathing Room Rule

Every section needs more space than you think it does. Premium products breathe.

```tsx
{/* ✅ Premium spacing */}
<section className="py-24 md:py-32 px-6 md:px-8">

{/* ❌ Cramped — looks like a template */}
<section className="py-8 px-4">
```

### Container Rules

```tsx
{/* Standard page container */}
<div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">

{/* Narrow content (articles, forms) */}
<div className="max-w-2xl mx-auto px-4 sm:px-6">

{/* Full-bleed with inner constraint */}
<section className="w-full bg-gradient-to-br from-white/4 to-transparent">
  <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-24">
```

### Grid Rules

Never use a uniform 3-column grid for feature cards — it reads like a Bootstrap template. Use asymmetry:

```tsx
{/* ✅ Bento grid — varied sizes */}
<div className="grid grid-cols-12 gap-4">
  <div className="col-span-12 md:col-span-8"> {/* large feature */} </div>
  <div className="col-span-12 md:col-span-4"> {/* tall sidebar */} </div>
  <div className="col-span-12 md:col-span-4"> {/* small */} </div>
  <div className="col-span-12 md:col-span-4"> {/* small */} </div>
  <div className="col-span-12 md:col-span-4"> {/* small */} </div>
</div>

{/* ✅ Asymmetric two-column */}
<div className="grid grid-cols-1 lg:grid-cols-[1fr_1.5fr] gap-16 items-center">

{/* ❌ Boring uniform grid */}
<div className="grid grid-cols-3 gap-6">
```

### Internal Card Spacing

Cards should feel spacious internally. Minimum `p-6`, preferred `p-8` for feature cards.

```tsx
{/* ✅ Generous */}
<div className="p-8 space-y-4">

{/* ❌ Cramped */}
<div className="p-3">
```

---

## 9. Component Rules

### Hero Section — Required Pattern

Every hero must have all six elements:

```tsx
<section className="relative min-h-[85vh] flex items-center justify-center
                    overflow-hidden px-6 py-32">

  {/* 1. Atmospheric background */}
  <div className="absolute inset-0 -z-10">
    <div className="absolute top-0 left-1/2 -translate-x-1/2 w-[900px] h-[700px]
                    bg-[#D4AF37]/10 rounded-full blur-[140px]" />
  </div>

  {/* 2. Optional: Globe, 3D, or full-bleed image */}
  {/* 3. Eyebrow label */}
  <div className="flex items-center gap-2 text-[#D4AF37] text-sm font-semibold
                  tracking-widest uppercase mb-6">
    <span className="w-8 h-px bg-[#D4AF37]" />
    Global Prayer Network
  </div>

  {/* 4. Headline — gold accent word */}
  <h1 className="font-display text-5xl md:text-7xl font-bold leading-[1.1]
                 tracking-tight text-center max-w-4xl">
    The World is{' '}
    <span className="bg-gradient-to-r from-[#D4AF37] via-[#E8CC6A] to-[#D4AF37]
                     bg-clip-text text-transparent">
      Praying
    </span>
  </h1>

  {/* 5. Sub-headline */}
  <p className="mt-6 text-lg md:text-xl text-[#94A3B8] max-w-2xl text-center
                leading-relaxed">
    Join 120,000 intercessors from 180 nations lifting prayers in unity.
  </p>

  {/* 6. CTA group */}
  <div className="mt-10 flex flex-wrap gap-4 justify-center">
    <PrimaryButton>Start Praying</PrimaryButton>
    <SecondaryButton>Explore the Globe</SecondaryButton>
  </div>

  {/* 7. Social proof / stat bar (optional but premium) */}
  <div className="mt-16 flex items-center gap-8 text-sm text-[#64748B]">
    <span><strong className="text-[#F8FAFC]">120K+</strong> Intercessors</span>
    <span className="w-px h-4 bg-white/10" />
    <span><strong className="text-[#F8FAFC]">2.4M+</strong> Prayers</span>
    <span className="w-px h-4 bg-white/10" />
    <span><strong className="text-[#F8FAFC]">180</strong> Nations</span>
  </div>

</section>
```

### Stat / Metric Cards — Required Pattern

```tsx
<div className="relative overflow-hidden rounded-2xl p-8
                bg-gradient-to-br from-white/8 to-white/2
                border border-white/8 backdrop-blur-xl">

  {/* Decorative glow */}
  <div className="absolute -top-10 -right-10 w-32 h-32
                  bg-[#D4AF37]/10 rounded-full blur-2xl pointer-events-none" />

  <div className="relative z-10">
    {/* Icon */}
    <div className="w-10 h-10 rounded-xl bg-[#D4AF37]/10 border border-[#D4AF37]/20
                    flex items-center justify-center mb-4">
      <Icon className="w-5 h-5 text-[#D4AF37]" />
    </div>

    {/* Big number */}
    <div className="text-4xl font-bold font-display text-[#F8FAFC] mb-1">
      <AnimatedNumber value={2400000} />+
    </div>

    {/* Label */}
    <div className="text-sm text-[#94A3B8]">Total Prayers</div>

    {/* Trend indicator */}
    <div className="mt-3 flex items-center gap-1 text-xs text-emerald-400">
      <TrendingUpIcon className="w-3.5 h-3.5" />
      +12% this week
    </div>
  </div>
</div>
```

### Empty State — Required Pattern

Empty states must be visually compelling — they are an opportunity, not an absence.

```tsx
<div className="flex flex-col items-center justify-center py-24 px-6 text-center">

  {/* Illustrated icon with glow */}
  <div className="relative mb-8">
    <div className="absolute inset-0 bg-[#D4AF37]/20 rounded-full blur-2xl scale-150" />
    <div className="relative w-24 h-24 rounded-full
                    bg-gradient-to-br from-[#D4AF37]/20 to-[#D4AF37]/5
                    border border-[#D4AF37]/20
                    flex items-center justify-center">
      <HandsIcon className="w-10 h-10 text-[#D4AF37]" />
    </div>
  </div>

  <h3 className="text-xl font-semibold text-[#F8FAFC] mb-2">
    No prayers yet
  </h3>
  <p className="text-[#94A3B8] max-w-sm leading-relaxed mb-8">
    Be the first to lift a prayer. Your request will reach thousands of
    intercessors across the world.
  </p>

  <PrimaryButton>Post a Prayer Request</PrimaryButton>

</div>
```

### Section Dividers

Never use `<hr>` or `border-t` as a visual separator between page sections. Use space, gradient fades, or decorative elements.

```tsx
{/* ✅ Atmospheric fade-in for next section */}
<div className="h-px w-full bg-gradient-to-r
                from-transparent via-white/10 to-transparent my-16" />

{/* ✅ Decorative section opener */}
<div className="flex items-center gap-4 mb-12">
  <div className="flex-1 h-px bg-gradient-to-r from-transparent to-white/10" />
  <span className="text-[#D4AF37] text-xs tracking-widest uppercase font-semibold">
    Answered Prayers
  </span>
  <div className="flex-1 h-px bg-gradient-to-l from-transparent to-white/10" />
</div>
```

### Badge / Tag Components

```tsx
{/* Category badge */}
<span className="inline-flex items-center gap-1.5 px-3 py-1 rounded-full
                 text-xs font-semibold tracking-wide
                 bg-[#D4AF37]/10 text-[#D4AF37] border border-[#D4AF37]/20">
  Healing
</span>

{/* Urgency: Critical */}
<span className="inline-flex items-center gap-1.5 px-3 py-1 rounded-full
                 text-xs font-semibold
                 bg-red-500/10 text-red-400 border border-red-500/20">
  <span className="w-1.5 h-1.5 rounded-full bg-red-400 animate-pulse" />
  Critical
</span>

{/* Live indicator */}
<span className="inline-flex items-center gap-1.5 px-3 py-1 rounded-full
                 text-xs font-semibold
                 bg-emerald-500/10 text-emerald-400 border border-emerald-500/20">
  <span className="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse" />
  Live
</span>
```

---

## 10. Loading & Empty States

### Loading States — Never Use a Bare Spinner

Every loading state must use shimmer skeletons that match the shape of the content they replace.

```tsx
function PrayerCardSkeleton() {
  return (
    <div className="rounded-2xl p-6 bg-white/4 border border-white/6 animate-pulse">
      <div className="flex items-center gap-3 mb-4">
        <div className="w-10 h-10 rounded-full bg-white/8" />
        <div className="space-y-2 flex-1">
          <div className="h-3 bg-white/8 rounded-full w-1/3" />
          <div className="h-2 bg-white/5 rounded-full w-1/4" />
        </div>
      </div>
      <div className="space-y-2">
        <div className="h-3 bg-white/8 rounded-full" />
        <div className="h-3 bg-white/8 rounded-full w-4/5" />
        <div className="h-3 bg-white/8 rounded-full w-2/3" />
      </div>
    </div>
  );
}
```

Render 3–6 skeleton cards while loading. They must maintain the same dimensions as real cards.

### Page-Level Loading

For full page loads, use a centered animated gold cross or logo mark — never a generic spinner:

```tsx
<div className="flex items-center justify-center min-h-screen">
  <motion.div
    animate={{ opacity: [0.3, 1, 0.3] }}
    transition={{ duration: 2, repeat: Infinity, ease: 'easeInOut' }}
  >
    <Logo className="w-12 h-12 text-[#D4AF37]" />
  </motion.div>
</div>
```

### Error States

Never show a raw error message. Wrap all errors in a contextual, actionable design:

```tsx
<div className="rounded-2xl p-8 bg-red-500/5 border border-red-500/15
                flex flex-col items-center text-center">
  <div className="w-12 h-12 rounded-full bg-red-500/10 flex items-center justify-center mb-4">
    <AlertCircle className="w-6 h-6 text-red-400" />
  </div>
  <h3 className="text-[#F8FAFC] font-semibold mb-2">Something went wrong</h3>
  <p className="text-[#94A3B8] text-sm mb-6">
    We couldn't load your prayer wall. Check your connection and try again.
  </p>
  <SecondaryButton onClick={retry}>Try Again</SecondaryButton>
</div>
```

---

## 11. Mobile Rules

### Mobile Is Not a Smaller Desktop

On screens below `md` (768px), layouts must be purpose-built for thumbs, not shrunk from desktop.

### Critical Mobile Patterns

```tsx
{/* Hero: tighten spacing, reduce text size */}
<section className="py-20 md:py-32 px-4 md:px-8">
  <h1 className="text-4xl md:text-7xl font-bold leading-[1.15] md:leading-[1.1]">

{/* Cards: always full-width on mobile, grid from md */}
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4 md:gap-6">

{/* CTA stack on mobile */}
<div className="flex flex-col sm:flex-row gap-3">
  <PrimaryButton className="w-full sm:w-auto">Join Now</PrimaryButton>
  <SecondaryButton className="w-full sm:w-auto">Learn More</SecondaryButton>
</div>
```

### Bottom Navigation (mobile only)

The bottom tab bar must float above content with a blur treatment — never sit flat:

```tsx
<nav className="fixed bottom-0 inset-x-0 z-50 md:hidden
                bg-[#0F172A]/90 backdrop-blur-2xl
                border-t border-white/8
                pb-[env(safe-area-inset-bottom)]">
  <div className="flex items-center justify-around h-16 px-2">
    {tabs.map((tab) => (
      <TabItem key={tab.id} {...tab} />
    ))}
  </div>
</nav>
```

### Touch Targets

Every tap target: minimum `44px × 44px`. For icon-only buttons, pad aggressively:

```tsx
<button className="p-3"> {/* 24px icon + 12px padding each side = 48px */}
  <Icon className="w-6 h-6" />
</button>
```

### Mobile Glass Performance

On mobile, cap `backdrop-filter`:

```tsx
<div className="backdrop-blur-lg md:backdrop-blur-xl"> {/* 12px mobile, 24px desktop */}
```

---

## 12. Anti-Patterns — Never Do These

### Layout Anti-Patterns

```tsx
{/* ❌ Flat solid background */}
<div className="bg-[#0F172A]">

{/* ❌ Uniform card grid (bootstrap smell) */}
<div className="grid grid-cols-3 gap-4">
  <div className="bg-gray-800 p-4 rounded">...</div>
  <div className="bg-gray-800 p-4 rounded">...</div>
  <div className="bg-gray-800 p-4 rounded">...</div>
</div>

{/* ❌ Section with no visual treatment */}
<section>
  <h2>Prayer Wall</h2>
  <div className="mt-4 space-y-2">

{/* ❌ Content directly on background — no card/surface */}
<div className="text-white p-4">
  <h2>Today's Prayers</h2>
```

### Component Anti-Patterns

```tsx
{/* ❌ Plain border-t divider between sections */}
<hr className="border-t border-gray-700 my-8" />

{/* ❌ Bare spinner loading state */}
{isLoading && <div className="animate-spin w-8 h-8 border-2 rounded-full" />}

{/* ❌ Unstyled empty state */}
{items.length === 0 && <p>No items found.</p>}

{/* ❌ Default browser form styles */}
<input type="text" className="border p-2" />

{/* ❌ Static numbers that should animate */}
<span className="text-4xl font-bold">120,000</span>

{/* ❌ Generic placeholder text */}
<p>Lorem ipsum dolor sit amet...</p>
```

### Animation Anti-Patterns

```tsx
{/* ❌ All elements animate simultaneously (no stagger) */}
{items.map(item => <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }}>)}

{/* ❌ Animating width/height (causes layout thrash) */}
<motion.div animate={{ width: isOpen ? 300 : 0 }}>

{/* ❌ CSS transition on card hover instead of Framer Motion */}
<div className="transition-transform hover:-translate-y-1">

{/* ❌ No AnimatePresence on conditional render */}
{isOpen && <Modal />}
```

### Typography Anti-Patterns

```tsx
{/* ❌ Full paragraph with gold gradient */}
<p className="bg-gradient-to-r from-[#D4AF37] to-[#E8CC6A] bg-clip-text text-transparent">
  Join 120,000 believers across the world...
</p>

{/* ❌ Text without max-width (unreadable on wide screens) */}
<p className="text-[#94A3B8]">A very long paragraph with no width constraint...</p>

{/* ❌ Heading and body at the same weight/size */}
<h2 className="text-base font-medium">Section Title</h2>
<p className="text-base font-medium">Body text here.</p>
```

---

## 13. Page-Type Patterns

### Landing / Marketing Page

1. Full-height hero with atmospheric glow + globe or 3D visual
2. Animated stats bar (`120K Intercessors | 2.4M Prayers | 180 Nations`)
3. Feature bento grid — not uniform columns
4. Testimonial/social proof section with avatar clusters
5. Globe or map visualisation section
6. Subscription/CTA section with gold glass card
7. Footer with newsletter + navigation

### Feed Page (Prayer Wall, Video Feed)

1. Sticky header with filter tabs
2. Feed container with max-width constraint
3. Skeleton cards on load → real cards with stagger
4. Infinite scroll with intersection observer
5. Floating compose button (bottom-right, gold, pulsing)
6. No empty white space between cards — subtle `border-b border-white/5` separator

### Profile Page

1. Full-bleed cover image with gradient overlay
2. Avatar overlapping the cover (negative margin)
3. Stats row (prayers, testimonies, streak) with animated numbers
4. Tab navigation below stats (Prayers | Testimonies | Groups)
5. Glass panel for bio/denomination/location
6. Badge row with tooltips

### Detail Page (Prayer Request, Testimony, Event)

1. Contextual header (back button, share, action menu)
2. Author row with avatar, name, date, category badge
3. Content area with generous line height
4. Engagement bar (pray count, comments, share)
5. Comments section with input pinned to bottom on mobile
6. Related items grid at the bottom

### Settings / Form Page

1. Left sidebar navigation (desktop) / top tabs (mobile)
2. Each settings section in its own glass card
3. Destructive actions (delete account) at the very bottom, in a red-tinted glass card
4. Save button fixed to bottom of viewport on mobile
5. Success/error states inline — never a page reload

### Dashboard (Admin / Creator)

1. Stats row at top with animated metric cards
2. Primary chart (full width) below stats
3. Two-column layout: main content left, activity feed right
4. Table with glass row hover states (never striped rows)
5. Filter bar with glass pill buttons

---

## 14. Quick-Reference Checklist

Before submitting any page implementation, verify every item:

**Atmosphere**
- [ ] Background has at least two radial glow elements
- [ ] Noise texture overlay present
- [ ] No flat solid `#0F172A` background without atmosphere layers

**Typography**
- [ ] One dominant display headline above the fold
- [ ] Gold gradient applied to at most one key phrase
- [ ] All body text constrained to `max-w-prose` or narrower
- [ ] Hierarchy is immediately clear (size + weight differentiation)

**Surfaces**
- [ ] All cards use glassmorphism (`backdrop-blur`, gradient fill, white border)
- [ ] Cards have decorative inner glow element
- [ ] No flat, opaque, unstyled containers

**Animation**
- [ ] Page sections use `whileInView` fade-up entrance
- [ ] List items stagger on entrance
- [ ] All interactive cards use `whileHover` lift
- [ ] Primary CTA has gold glow pulse animation
- [ ] Modals use `AnimatePresence` + `scaleIn` variant
- [ ] Stat numbers use `AnimatedNumber` counter
- [ ] `useReducedMotion()` respected

**States**
- [ ] Loading state: shimmer skeleton (not spinner)
- [ ] Empty state: illustrated icon + headline + CTA
- [ ] Error state: contextual message + retry action

**Mobile**
- [ ] All touch targets ≥ 44px
- [ ] Bottom tab bar present and floats with blur
- [ ] Modals convert to bottom-sheet drawers
- [ ] `backdrop-blur` capped at `blur-lg` on mobile
- [ ] Safe area insets applied to bottom-fixed elements

**Spacing**
- [ ] Section vertical padding: `py-20 md:py-32` minimum
- [ ] Cards: `p-6` minimum internal padding
- [ ] No two consecutive sections share the same layout pattern

**Never**
- [ ] No `<hr>` or flat `border-t` section dividers
- [ ] No uniform 3-column card grids
- [ ] No bare text on background (always on a glass surface or in a defined container)
- [ ] No static counters on stats that should animate
- [ ] No unstyled empty states or bare loading spinners
- [ ] No gold gradient on more than one phrase per page
