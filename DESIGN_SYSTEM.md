# DESIGN_SYSTEM.md

**120 Army — Global Christian Prayer & Discipleship Platform**

A premium, dark-mode-first design system. Every decision prioritises spiritual gravitas, modern craft, and human warmth. Inspiration: Apple, Linear, Stripe, Arc Browser, Notion.

---

## 1. Color Palette

### Core

| Token | Hex | Usage |
|---|---|---|
| `gold-primary` | `#D4AF37` | CTAs, highlights, active states, brand moments |
| `gold-light` | `#E8CC6A` | Hover states on gold, icon fills |
| `gold-muted` | `#A8891E` | Pressed states, secondary gold text |
| `navy-bg` | `#0F172A` | Base page background |
| `navy-surface` | `#1E293B` | Card backgrounds, panel surfaces |
| `navy-elevated` | `#263348` | Elevated layers, modals, dropdowns |
| `navy-border` | `#334155` | Subtle borders, dividers |
| `accent-blue` | `#3B82F6` | Links, informational states, secondary actions |
| `accent-blue-light` | `#60A5FA` | Hover on blue, icon accent |

### Semantic

| Token | Hex | Usage |
|---|---|---|
| `success` | `#22C55E` | Confirmation, answered prayers |
| `warning` | `#F59E0B` | Caution, pending |
| `error` | `#EF4444` | Destructive actions, alerts |
| `text-primary` | `#F8FAFC` | Headings, body copy |
| `text-secondary` | `#94A3B8` | Captions, metadata, placeholders |
| `text-muted` | `#475569` | Disabled text, timestamps |

### Gradient Presets

```css
/* Gold shimmer — hero headings, premium moments */
--gradient-gold: linear-gradient(135deg, #D4AF37 0%, #E8CC6A 50%, #A8891E 100%);

/* Deep spiritual — full-page hero backgrounds */
--gradient-hero: radial-gradient(ellipse at top, #1E293B 0%, #0F172A 60%, #0A0F1E 100%);

/* Blue pulse — live/active indicators */
--gradient-pulse: linear-gradient(135deg, #3B82F6 0%, #6366F1 100%);

/* Glass overlay — glassmorphic surfaces */
--gradient-glass: linear-gradient(135deg, rgba(255,255,255,0.08) 0%, rgba(255,255,255,0.02) 100%);
```

---

## 2. Typography Scale

**Font stack**

```css
--font-display: 'Cal Sans', 'SF Pro Display', system-ui, sans-serif;  /* headings */
--font-body:    'Inter', 'SF Pro Text', system-ui, sans-serif;         /* body copy */
--font-mono:    'JetBrains Mono', 'SF Mono', monospace;                /* code, IDs */
```

### Scale

| Token | Size | Line Height | Weight | Usage |
|---|---|---|---|---|
| `display-2xl` | 72px / 4.5rem | 1.1 | 700 | Hero headlines |
| `display-xl` | 56px / 3.5rem | 1.15 | 700 | Section heroes |
| `display-lg` | 40px / 2.5rem | 1.2 | 600 | Page titles |
| `heading-xl` | 32px / 2rem | 1.25 | 600 | Card titles, modal headers |
| `heading-lg` | 24px / 1.5rem | 1.3 | 600 | Section headings |
| `heading-md` | 20px / 1.25rem | 1.4 | 600 | Sub-headings |
| `heading-sm` | 16px / 1rem | 1.4 | 600 | Labels, form section titles |
| `body-lg` | 18px / 1.125rem | 1.6 | 400 | Lead paragraphs |
| `body-md` | 16px / 1rem | 1.6 | 400 | Default body |
| `body-sm` | 14px / 0.875rem | 1.5 | 400 | Secondary text |
| `caption` | 12px / 0.75rem | 1.4 | 400 | Metadata, timestamps |
| `overline` | 11px / 0.6875rem | 1.4 | 600 | ALL CAPS labels, category tags |

### Rules

- Headings use `--font-display`; body uses `--font-body`
- Letter-spacing on display sizes: `−0.02em`; on overline: `+0.1em`
- Never set body text below 14px
- Gold gradient text (`--gradient-gold`) reserved for hero headlines only — not for body copy

---

## 3. Spacing System

Base unit: **4px (0.25rem)**. All spacing uses multiples of this base.

| Token | Value | Common Usage |
|---|---|---|
| `space-1` | 4px | Icon gaps, tight inline spacing |
| `space-2` | 8px | Between label and input, icon padding |
| `space-3` | 12px | Compact list items |
| `space-4` | 16px | Default internal padding |
| `space-5` | 20px | Form field gaps |
| `space-6` | 24px | Card internal padding |
| `space-8` | 32px | Section element spacing |
| `space-10` | 40px | Between card groups |
| `space-12` | 48px | Section top/bottom padding |
| `space-16` | 64px | Large section separation |
| `space-20` | 80px | Hero vertical padding |
| `space-24` | 96px | Page-level section gaps |
| `space-32` | 128px | Maximum hero padding |

**Layout containers**

```css
--container-sm:  640px;
--container-md:  768px;
--container-lg:  1024px;
--container-xl:  1280px;
--container-2xl: 1440px;

--page-padding-x: clamp(1rem, 5vw, 6rem);  /* responsive horizontal page margin */
```

---

## 4. Border Radius

| Token | Value | Usage |
|---|---|---|
| `radius-sm` | 4px | Badges, chips, small tags |
| `radius-md` | 8px | Inputs, small buttons |
| `radius-lg` | 12px | Cards, panels |
| `radius-xl` | 16px | Modals, large cards |
| `radius-2xl` | 24px | Feature sections, large modals |
| `radius-3xl` | 32px | Hero containers, full-bleed sections |
| `radius-full` | 9999px | Pills, avatars, icon buttons |

**Rule:** Never mix sharp (0px) corners with highly rounded elements on the same surface. Maintain radius consistency within a card group.

---

## 5. Shadow System

All shadows are tuned for dark backgrounds — they use glow, not traditional drop shadows.

```css
/* Subtle depth */
--shadow-sm:  0 1px 3px rgba(0,0,0,0.4), 0 1px 2px rgba(0,0,0,0.3);

/* Default card elevation */
--shadow-md:  0 4px 16px rgba(0,0,0,0.5), 0 2px 4px rgba(0,0,0,0.3);

/* Raised/floating cards */
--shadow-lg:  0 8px 32px rgba(0,0,0,0.6), 0 4px 8px rgba(0,0,0,0.4);

/* Modals, overlays */
--shadow-xl:  0 20px 60px rgba(0,0,0,0.7), 0 8px 24px rgba(0,0,0,0.5);

/* Gold glow — primary CTA, active/focus states */
--shadow-gold: 0 0 20px rgba(212,175,55,0.35), 0 0 60px rgba(212,175,55,0.15);

/* Blue glow — accent, informational, links */
--shadow-blue: 0 0 20px rgba(59,130,246,0.35), 0 0 60px rgba(59,130,246,0.15);

/* Inset — pressed button state */
--shadow-inset: inset 0 2px 8px rgba(0,0,0,0.4);
```

---

## 6. Glassmorphism Rules

Glassmorphism is the primary surface treatment. Apply it consistently.

### Standard Glass Surface

```css
.glass {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.08) 0%,
    rgba(255, 255, 255, 0.02) 100%
  );
  backdrop-filter: blur(20px) saturate(180%);
  -webkit-backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: var(--radius-lg);
}
```

### Elevated Glass (modals, dropdowns)

```css
.glass-elevated {
  background: linear-gradient(
    135deg,
    rgba(255, 255, 255, 0.12) 0%,
    rgba(255, 255, 255, 0.04) 100%
  );
  backdrop-filter: blur(40px) saturate(200%);
  -webkit-backdrop-filter: blur(40px) saturate(200%);
  border: 1px solid rgba(255, 255, 255, 0.12);
}
```

### Gold Glass (featured/highlighted surfaces)

```css
.glass-gold {
  background: linear-gradient(
    135deg,
    rgba(212, 175, 55, 0.12) 0%,
    rgba(212, 175, 55, 0.04) 100%
  );
  backdrop-filter: blur(20px) saturate(180%);
  border: 1px solid rgba(212, 175, 55, 0.2);
  box-shadow: var(--shadow-gold);
}
```

### Rules

- Always place glass surfaces over a non-flat background (gradient, image, or layered color) — glass on solid colour is invisible
- Minimum blur: `16px`; maximum: `40px`
- Border opacity should stay between `0.06–0.15` to avoid harsh edges
- Do not stack more than 3 glass layers — depth becomes unreadable
- On mobile, reduce `backdrop-filter` blur to `12px` for performance

---

## 7. Animation Rules

All animations use **Framer Motion**. No raw CSS `transition` on interactive elements unless it's a simple `opacity` or `color` hover.

### Duration Tokens

```ts
export const duration = {
  instant:  0.1,   // micro-interactions (hover glow)
  fast:     0.2,   // button press, toggle
  normal:   0.3,   // default transitions
  smooth:   0.5,   // card entrance, modal open
  slow:     0.8,   // hero text reveals
  crawl:    1.2,   // background drifts, globe rotations
};
```

### Easing Tokens

```ts
export const ease = {
  out:    [0.0, 0.0, 0.2, 1.0],   // decelerate — entering elements
  in:     [0.4, 0.0, 1.0, 1.0],   // accelerate — exiting elements
  inOut:  [0.4, 0.0, 0.2, 1.0],   // balanced — state changes
  spring: { type: 'spring', stiffness: 300, damping: 30 },  // playful, physical
  gentle: { type: 'spring', stiffness: 120, damping: 20 },  // soft reveals
};
```

### Standard Variants

```ts
// Fade up — default for cards, sections
export const fadeUp = {
  hidden:  { opacity: 0, y: 24 },
  visible: { opacity: 1, y: 0, transition: { duration: 0.5, ease: ease.out } },
};

// Scale in — modals, popovers
export const scaleIn = {
  hidden:  { opacity: 0, scale: 0.95 },
  visible: { opacity: 1, scale: 1, transition: { duration: 0.3, ease: ease.out } },
  exit:    { opacity: 0, scale: 0.95, transition: { duration: 0.2, ease: ease.in } },
};

// Stagger children — lists, grids
export const staggerContainer = {
  hidden:  {},
  visible: { transition: { staggerChildren: 0.08, delayChildren: 0.1 } },
};

// Slide in from right — drawers, panels
export const slideInRight = {
  hidden:  { opacity: 0, x: 40 },
  visible: { opacity: 1, x: 0, transition: { duration: 0.4, ease: ease.out } },
  exit:    { opacity: 0, x: 40, transition: { duration: 0.25, ease: ease.in } },
};
```

### Rules

- Every page section uses `fadeUp` with stagger on list items
- `AnimatePresence` wraps all conditional renders (modals, toasts, route changes)
- Respect `prefers-reduced-motion` — wrap animation values in `useReducedMotion()`
- Never animate `width` or `height` directly — use `scaleX`/`scaleY` or `layout` prop
- Globe and Three.js scenes animate on a separate RAF loop — do not mix with Framer Motion

---

## 8. Button Variants

### Primary (Gold)

```tsx
// bg: gold gradient | text: navy | glow on hover
className="
  inline-flex items-center justify-center gap-2
  px-6 py-3 rounded-full
  bg-gradient-to-r from-[#D4AF37] to-[#E8CC6A]
  text-[#0F172A] font-semibold text-sm tracking-wide
  shadow-[0_0_20px_rgba(212,175,55,0.3)]
  hover:shadow-[0_0_32px_rgba(212,175,55,0.5)] hover:scale-[1.02]
  active:scale-[0.98]
  transition-all duration-200
  focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[#D4AF37]
"
```

### Secondary (Ghost)

```tsx
// glass surface | gold border | gold text
className="
  inline-flex items-center justify-center gap-2
  px-6 py-3 rounded-full
  bg-white/5 backdrop-blur-md
  border border-[#D4AF37]/30
  text-[#D4AF37] font-semibold text-sm
  hover:bg-white/10 hover:border-[#D4AF37]/60
  active:scale-[0.98]
  transition-all duration-200
"
```

### Tertiary (Text)

```tsx
// no background | subtle text | underline on hover
className="
  inline-flex items-center gap-1
  text-[#94A3B8] text-sm font-medium
  hover:text-[#F8FAFC]
  transition-colors duration-150
  underline-offset-4 hover:underline
"
```

### Destructive

```tsx
className="
  inline-flex items-center justify-center gap-2
  px-6 py-3 rounded-full
  bg-red-500/10 border border-red-500/30
  text-red-400 font-semibold text-sm
  hover:bg-red-500/20 hover:border-red-500/50
  transition-all duration-200
"
```

### Icon Button

```tsx
className="
  p-2.5 rounded-full
  bg-white/5 border border-white/8
  text-[#94A3B8] hover:text-[#F8FAFC]
  hover:bg-white/10
  transition-all duration-150
"
```

### Sizes

| Size | Padding | Font | Radius |
|---|---|---|---|
| `xs` | `px-3 py-1.5` | 12px | `rounded-full` |
| `sm` | `px-4 py-2` | 13px | `rounded-full` |
| `md` (default) | `px-6 py-3` | 14px | `rounded-full` |
| `lg` | `px-8 py-4` | 16px | `rounded-full` |
| `xl` | `px-10 py-5` | 18px | `rounded-full` |

---

## 9. Input Styles

### Default Input

```tsx
className="
  w-full px-4 py-3
  rounded-xl
  bg-white/5 backdrop-blur-sm
  border border-white/10
  text-[#F8FAFC] placeholder-[#475569]
  text-sm font-normal
  outline-none
  focus:border-[#D4AF37]/60 focus:bg-white/8 focus:ring-2 focus:ring-[#D4AF37]/20
  hover:border-white/20
  transition-all duration-200
"
```

### Textarea

Same as input; set `resize-none` and `min-h-[120px]`.

### Select

Replicate input styles. Use a custom chevron icon — never the browser default arrow.

### Checkbox / Radio

Use shadcn/ui primitives with overridden accent colour `#D4AF37`.

### Form Field Anatomy

```
[Label — body-sm, text-secondary, mb-1.5]
[Input]
[Helper text — caption, text-muted, mt-1.5]   ← or →   [Error — caption, text-error, mt-1.5]
```

### States

| State | Border | Ring |
|---|---|---|
| Default | `white/10` | none |
| Hover | `white/20` | none |
| Focus | `gold/60` | `gold/20` 2px |
| Error | `red-500/60` | `red-500/20` 2px |
| Disabled | `white/5` | none, `opacity-40` |

---

## 10. Card Styles

### Base Card

```tsx
className="
  relative overflow-hidden
  rounded-2xl p-6
  bg-gradient-to-br from-white/8 to-white/2
  backdrop-blur-xl
  border border-white/8
  shadow-[0_8px_32px_rgba(0,0,0,0.5)]
  hover:border-white/14 hover:shadow-[0_12px_40px_rgba(0,0,0,0.6)]
  hover:-translate-y-1
  transition-all duration-300
"
```

### Featured / Highlighted Card (gold border)

```tsx
className="
  relative overflow-hidden
  rounded-2xl p-6
  bg-gradient-to-br from-[#D4AF37]/10 to-[#D4AF37]/2
  backdrop-blur-xl
  border border-[#D4AF37]/25
  shadow-[0_8px_32px_rgba(0,0,0,0.5),0_0_40px_rgba(212,175,55,0.1)]
"
```

### Stat / Metric Card

Compact card (`p-4`, `rounded-xl`) with a large number in `display-xl` gold, label in `caption` text-secondary, and a micro sparkline or icon.

### Rules

- Cards should float — always include `hover:-translate-y-1` or an equivalent lift
- The inner top-right of a card can carry a decorative radial glow: `absolute -top-12 -right-12 w-40 h-40 bg-[#D4AF37]/10 rounded-full blur-3xl pointer-events-none`
- Never lay cards in uniform grids without variation in size or emphasis

---

## 11. Modal Styles

```tsx
/* Overlay */
className="fixed inset-0 bg-black/70 backdrop-blur-sm z-50"

/* Panel */
className="
  fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2
  w-full max-w-lg max-h-[90vh] overflow-y-auto
  rounded-3xl p-8
  bg-gradient-to-br from-[#1E293B] to-[#0F172A]
  border border-white/10
  shadow-[0_24px_80px_rgba(0,0,0,0.8)]
  z-50
"
```

### Anatomy

```
[Close button — top-right, icon button]
[Title — heading-xl]
[Subtitle — body-sm, text-secondary]
[Divider — border-t border-white/8, my-6]
[Content area]
[Footer — flex justify-end gap-3, pt-6 border-t border-white/8]
  [Cancel — tertiary button]
  [Confirm — primary button]
```

### Drawer (mobile modal)

On `< md` breakpoint, modals convert to bottom-sheet drawers:

```tsx
className="
  fixed bottom-0 left-0 right-0
  rounded-t-3xl p-6
  bg-[#1E293B] border-t border-white/10
  max-h-[90vh] overflow-y-auto
"
```

Include a drag handle: `<div className="w-10 h-1 rounded-full bg-white/20 mx-auto mb-6" />`

---

## 12. Navigation Styles

### Top Nav (desktop)

```tsx
className="
  fixed top-0 inset-x-0 z-40
  flex items-center justify-between
  px-8 h-16
  bg-[#0F172A]/80 backdrop-blur-xl
  border-b border-white/6
"
```

- Logo left, nav links centre, actions right
- Active link: gold colour + `border-b-2 border-[#D4AF37]`
- Inactive link: `text-secondary`, hover `text-primary`, `duration-150`

### Sidebar (app)

```tsx
className="
  fixed left-0 top-0 bottom-0 w-64
  flex flex-col
  bg-[#0F172A]/95 backdrop-blur-xl
  border-r border-white/6
  py-6 px-4
  z-30
"
```

- Active item: `bg-[#D4AF37]/10 text-[#D4AF37] border-l-2 border-[#D4AF37]`
- Inactive item: `text-[#94A3B8] hover:bg-white/5 hover:text-[#F8FAFC]`
- Use `rounded-r-xl` on items, `pl-3` for indent

### Bottom Tab Bar (mobile)

```tsx
className="
  fixed bottom-0 inset-x-0 z-40
  flex items-center justify-around
  h-16 px-4
  bg-[#0F172A]/95 backdrop-blur-2xl
  border-t border-white/8
"
```

- Active tab: gold icon + gold label (`text-xs font-semibold`)
- Inactive tab: muted icon, no label on very small screens

---

## 13. Mobile Design Rules

**Breakpoints (Tailwind)**

| Token | Width | Context |
|---|---|---|
| `sm` | 640px | Large phones (landscape) |
| `md` | 768px | Tablets |
| `lg` | 1024px | Small laptops |
| `xl` | 1280px | Desktop |
| `2xl` | 1440px | Wide desktop |

**Principles**

1. **Mobile-first** — write base styles for mobile, use `md:` / `lg:` to scale up
2. **Touch targets** — minimum 44×44px for all tappable elements
3. **Typography scaling** — hero headings scale down: `text-4xl md:text-5xl lg:text-7xl`
4. **Cards** — full-width on `< md`, grid from `md` up
5. **Modals** → bottom-sheet drawers on `< md` (see §11)
6. **Navigation** — sidebar collapses to bottom tab bar on `< md`; top nav hamburger opens full-screen drawer
7. **Glassmorphism** — reduce `backdrop-blur` to `12px` on mobile for performance; conditionally disable with CSS `@media (prefers-reduced-transparency: reduce)`
8. **Spacing** — horizontal page padding: `px-4` mobile → `px-6` tablet → `px-8+` desktop
9. **Animations** — reduce `y` offsets (24px → 12px) and duration (`0.5s` → `0.3s`) on mobile
10. **Safe areas** — use `pb-safe` / `pt-safe` (Tailwind `safe-area-inset-*`) on iPhone notch/home bar layouts

---

## Tailwind Config Reference

```ts
// tailwind.config.ts
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: 'class',
  content: ['./src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        gold: {
          DEFAULT: '#D4AF37',
          light:   '#E8CC6A',
          muted:   '#A8891E',
        },
        navy: {
          DEFAULT: '#0F172A',
          surface:  '#1E293B',
          elevated: '#263348',
          border:   '#334155',
        },
        accent: {
          DEFAULT: '#3B82F6',
          light:   '#60A5FA',
        },
      },
      fontFamily: {
        display: ['Cal Sans', 'SF Pro Display', 'system-ui', 'sans-serif'],
        body:    ['Inter', 'SF Pro Text', 'system-ui', 'sans-serif'],
        mono:    ['JetBrains Mono', 'SF Mono', 'monospace'],
      },
      borderRadius: {
        '3xl': '1.5rem',
        '4xl': '2rem',
      },
      backdropBlur: {
        xs: '4px',
      },
      animation: {
        'gold-pulse': 'goldPulse 3s ease-in-out infinite',
        'float':      'float 6s ease-in-out infinite',
      },
      keyframes: {
        goldPulse: {
          '0%, 100%': { boxShadow: '0 0 20px rgba(212,175,55,0.2)' },
          '50%':      { boxShadow: '0 0 40px rgba(212,175,55,0.5)' },
        },
        float: {
          '0%, 100%': { transform: 'translateY(0px)' },
          '50%':      { transform: 'translateY(-12px)' },
        },
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};

export default config;
```
