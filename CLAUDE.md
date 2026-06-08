# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mission

Build the world's leading Christian prayer, discipleship, and community platform — **120 Army**.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js (App Router) |
| Language | TypeScript (strict mode) |
| Styling | Tailwind CSS |
| Animation | Framer Motion |
| Components | shadcn/ui + Magic UI |
| 3D / Globe | Three.js, Globe.gl |

---

## Brand

**Colors**

- Primary (gold): `#D4AF37`
- Background (dark navy): `#0F172A`
- Accent (blue): `#3B82F6`

**Design language**

- Dark mode first
- Glassmorphism surfaces
- Floating cards with large spacing
- Premium typography
- Framer Motion animations on every meaningful interaction

**Inspiration:** Apple, Linear, Stripe, Airbnb, Arc Browser

**Never produce:** Bootstrap aesthetics, generic dashboards, repetitive card grids, or basic Tailwind layouts.

---

## Code Standards

- **TypeScript strict** — no `any`, no implicit types
- **Reusable, modular components** — no duplicated UI logic
- **Custom components only** — never use generic admin templates or placeholder UI
- **Production-ready output** — every commit should be shippable

---

## Component Requirements

Every page and feature must include:

- Responsive mobile layout
- Smooth Framer Motion animations
- Loading states
- Empty states
- Error states
- Accessibility support (semantic HTML, ARIA where needed)

---

## Development Commands

```bash
npm run dev        # start dev server
npm run build      # production build
npm run lint       # ESLint
npm run type-check # tsc --noEmit
```

---

## Architecture Notes

- Use Next.js App Router (`app/` directory) with React Server Components by default; opt into `"use client"` only when interactivity or browser APIs are needed.
- Tailwind config must include the brand color palette (`D4AF37`, `0F172A`, `3B82F6`) as named tokens.
- Framer Motion variants should be defined at the component level and shared via `AnimatePresence` for page transitions.
- Globe.gl and Three.js scenes are client-only — wrap in dynamic imports with `ssr: false`.
- shadcn/ui components should be customized to match the dark-mode-first brand theme, not used with default styles.
