# Gb-Technologies: Design System Specification

This specification establishes the atomic design foundation, responsive layout architecture, component states, and visual effects toolkit for the next-generation **Gb-Technologies** product and marketing platform.

---

## 1. Design Tokens Specification

### Spacing Scale (4px / 8px Base System)
All padding, margin, gap, and dimension values derive from an 8-point grid with 4-point micro-increments:

```css
:root {
  --space-3xs: 0.125rem; /* 2px - Hairline borders, sub-pixel alignments */
  --space-2xs: 0.25rem;  /* 4px - Micro-padding, badge insets, icon gap */
  --space-xs:  0.5rem;   /* 8px - Small button padding, chip spacing */
  --space-sm:  0.75rem;  /* 12px - Form field interior padding, card gap sm */
  --space-md:  1.0rem;   /* 16px - Base layout padding, standard card gap */
  --space-lg:  1.5rem;   /* 24px - Container gutter, section sub-grid */
  --space-xl:  2.0rem;   /* 32px - Section card padding, bento grid gap */
  --space-2xl: 3.0rem;   /* 48px - Section vertical rhythm mobile */
  --space-3xl: 4.0rem;   /* 64px - Section vertical rhythm desktop */
  --space-4xl: 6.0rem;   /* 96px - Major hero spacing, landing margins */
}
```

### Typography Scale & Fluid `clamp()` Formulas
- **Primary Typeface**: `Plus Jakarta Sans` or `Geist Sans` (high legibility, geometric precision, modern character).
- **Monospace Typeface**: `Geist Mono` or `JetBrains Mono` (metrics, code snippets, badge counters).
- **Measure Rule**: All long-form body copy enforces a strict measure of **60 to 75 characters per line** (`max-w-[65ch]`) to maximize reading velocity and minimize cognitive fatigue.
- **Headlines Tracking**: Tight tracking (`-0.03em` for display, `-0.02em` for H1/H2, `-0.01em` for H3) to create punchy, editorial impact.

| Token Name | Fluid CSS Clamp Formula | Line Height | Tracking | Desktop / Mobile Size |
| :--- | :--- | :--- | :--- | :--- |
| `--text-display` | `clamp(2.75rem, 5vw + 1.25rem, 5.0rem)` | 1.05 | `-0.035em` | 80px / 44px |
| `--text-h1` | `clamp(2.25rem, 3.5vw + 1rem, 3.75rem)` | 1.12 | `-0.030em` | 60px / 36px |
| `--text-h2` | `clamp(1.75rem, 2.5vw + 0.75rem, 2.75rem)`| 1.18 | `-0.025em` | 44px / 28px |
| `--text-h3` | `clamp(1.35rem, 1.5vw + 0.65rem, 2.0rem)` | 1.25 | `-0.020em` | 32px / 22px |
| `--text-h4` | `clamp(1.15rem, 0.8vw + 0.75rem, 1.5rem)` | 1.30 | `-0.015em` | 24px / 18px |
| `--text-body-lg` | `clamp(1.05rem, 0.4vw + 0.95rem, 1.25rem)`| 1.60 | `0` | 20px / 17px |
| `--text-body` | `clamp(0.95rem, 0.25vw + 0.9rem, 1.05rem)`| 1.65 | `0` | 16.8px / 15.2px |
| `--text-body-sm` | `clamp(0.85rem, 0.15vw + 0.8rem, 0.925rem)`| 1.50 | `+0.010em` | 14.8px / 13.6px |
| `--text-caption` | `0.75rem` (12px fixed) | 1.40 | `+0.025em` | 12px (Uppercase optional) |

### Corner Radius
```css
:root {
  --radius-none: 0px;
  --radius-sm:   6px;   /* Tags, badges, inner control elements */
  --radius-md:   10px;  /* Inputs, small buttons, dropdown menus */
  --radius-lg:   16px;  /* Standard cards, dialog modals, feature boxes */
  --radius-xl:   24px;  /* Bento hero cards, floating navbar container */
  --radius-full: 9999px;/* Pill buttons, avatar rings, live indicators */
}
```

### Motion & Transition Tokens
```css
:root {
  --ease-spring: cubic-bezier(0.16, 1, 0.3, 1); /* Natural deceleration without overshoot */
  --ease-snappy: cubic-bezier(0.2, 0.8, 0.2, 1); /* Instant feedback */
  --duration-fast:   150ms; /* Micro-states: hover, active, focus */
  --duration-base:   220ms; /* Tooltips, dropdown popovers, tab switching */
  --duration-smooth: 320ms; /* Modal entry, drawer slides, accordion expansion */
  --duration-enter:  500ms; /* Hero reveal, bento card staggered entrance */
}

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

---

## 2. Layout & Grid Principles

### Macro Layout (CSS Grid)
- **12-Column Fluid Grid**: Defined with `grid-template-columns: repeat(12, minmax(0, 1fr))` with fluid gutters (`--space-md` to `--space-xl`).
- **Centered Macro Container**: Maximum width capped at `1240px` (`max-w-[1240px]`), centered with `margin-inline: auto` and fluid padding (`clamp(1rem, 4vw, 2.5rem)`).
- **Asymmetric Bento Grids**: Using `grid-template-areas` or span utilities (`col-span-8`, `col-span-4`, `row-span-2`) to showcase product features in varying hierarchical sizes.

### Micro Layout (Flexbox & Container Queries)
- **Flexbox**: Used exclusively for interior component layouts (buttons with icons, form control groups, navigation links, metadata lists).
- **Container Queries (`@container`)**: Implemented on all modular cards (e.g. Project Showcase Card, Pricing Box). When placed in a narrow 4-column sidebar or a full 12-column row, the card responds to its **container width**, rearranging from vertical stack to horizontal layout automatically.

---

## 3. Component Inventory & 8-State Coverage Matrix

Every interactive component is required to have visual styling across **all 8 states**:

```
[ Default ] ──> [ Hover ] ──> [ Focus-Visible ] ──> [ Active / Pressed ]
      │
      ├──> [ Loading / Skeleton ]
      ├──> [ Disabled ]
      ├──> [ Error / Invalid ]
      └──> [ Success / Completed ]
```

### Core Primitives & Components

1. **Button**:
   - *Variants*: Primary (Gradient Glow), Secondary (Surface Outline), Ghost, Destructive, Link.
   - *States*: Default, Hover (`translate-y(-1px)`, tinted shadow bloom), Focus-visible (double ring `2px offset 2px`), Active (`translate-y(0.5px)`), Disabled (`opacity-40 cursor-not-allowed`), Loading (spinner + text preserve layout).
2. **Text Input & Textarea**:
   - Floating label or top label, leading/trailing icon slots.
   - States: Default, Hover, Focus (brand ring), Error (red border + shake + helper text), Success (green checkmark), Disabled.
3. **Bento Card (Interactive Surface)**:
   - Subtle gradient border, top-left radial glow, container query enabled.
   - Hover state: Ambient cursor spotlight highlight, subtle scale (`scale-[1.01]`).
4. **Header Navigation (Glass Navbar)**:
   - Floating pill or fixed blur header (`backdrop-blur-md`, `bg-canvas/75`).
   - Active section pill indicator with shared layout motion (`layoutId="navbar-pill"`).
5. **Interactive Filterable Portfolio Grid**:
   - Filter pills (All, Web Applications, Cloud Platforms, UI/UX Systems).
   - Animate layout shifts with Framer Motion `layout` prop.
6. **Accordion (FAQ)**:
   - Smooth height transition (`grid-template-rows: 0fr -> 1fr`).
   - Chevron rotation 180deg.
7. **Profile Drawer / Client Dashboard Panel**:
   - Slide-over sheet with focus trap, backdrop dim, and escape-key dismissal.

### Storybook Architecture
Organized according to Atomic Design principles:
```
storybook/
├── 01-Tokens/
│   ├── Colors.stories.tsx
│   ├── Typography.stories.tsx
│   ├── Spacing.stories.tsx
│   └── Shadows.stories.tsx
├── 02-Primitives/
│   ├── Button.stories.tsx
│   ├── Input.stories.tsx
│   ├── Badge.stories.tsx
│   └── Tooltip.stories.tsx
├── 03-Components/
│   ├── BentoCard.stories.tsx
│   ├── Navbar.stories.tsx
│   ├── PricingCard.stories.tsx
│   └── FAQAccordion.stories.tsx
├── 04-Marketing/
│   ├── HeroSection.stories.tsx
│   ├── ServicesBento.stories.tsx
│   ├── PortfolioGrid.stories.tsx
│   └── Footer.stories.tsx
└── 05-Portal/
    ├── ProfileOverview.stories.tsx
    └── EditProfileModal.stories.tsx
```

---

## 4. Style & Visual Effects Toolkit Evaluation

We evaluated 13 modern aesthetic treatments against usability, brand identity, and performance budgets:

| Effect / Style | Assessment & Verdict | Implementation Decision & Rules |
| :--- | :--- | :--- |
| **Bento Grids** | ✅ **PRIMARY STYLE** | Organizes complex features into modular, digestible visual chunks. Default layout for Services & Capabilities. |
| **Dark Neutral UI** | ✅ **PRIMARY PALETTE** | Deep tinted obsidian (`oklch(0.13 ...)`), not pitch black. Creates high contrast and luxury technical feel. |
| **Gradient Borders & Glow** | ✅ **KEY ACCENT** | `1px` subtle gradient borders with radial glow on cards. Maximum 1 glowing accent button per viewport. |
| **Subtle Grain/Noise** | ✅ **PERMITTED ACCENT** | 3% opacity SVG noise overlay on the hero canvas to eliminate banding on dark gradients. |
| **Glassmorphism / Liquid Glass**| ⚠️ **RESTRICTED** | Permitted ONLY on sticky navbar and dropdown menus. `backdrop-filter: blur(14px)`. Disabled on mobile if GPU lags. |
| **Aurora & Mesh Gradients** | ⚠️ **RESTRICTED** | Allowed only in Hero background at low opacity (12-18%). Static CSS blur; no CPU-heavy canvas animation. |
| **Cursor Spotlight Effect** | ⚠️ **DESKTOP ONLY** | Radial gradient tracking mouse coordinates on bento card borders. Automatically disabled on touch screens. |
| **Scroll-Driven Animation** | ⚠️ **TARGETED** | Native CSS `animation-timeline: view()` where supported, progressive enhancement for section fade-ins. |
| **Kinetic Typography** | ❌ **REJECTED** | Distracting and impairs readability. Substituted by clean stagger entrance via Framer Motion. |
| **Neumorphism** | ❌ **REJECTED** | Terrible accessibility contrast ratios and looks outdated for modern high-performance engineering. |
| **Claymorphism** | ❌ **REJECTED** | Inappropriate for a serious technical software and engineering firm. |
| **Neobrutalism** | ❌ **REJECTED** | High visual noise, clashes with premium SaaS/enterprise product aesthetics. |
| **Heavy 3D / WebGL** | ❌ **REJECTED** | Severe battery drain, heavy bundle (+500KB Three.js), and slow Core Web Vitals (LCP/INP penalty). |

### Performance Budget for Visual Effects
- **Backdrop Blur Limit**: Max `16px`. Restricted to `<header>` and modal overlays.
- **Paint Complexity**: Will not animate `filter: blur()` (causes GPU redraw spikes). Animate `opacity` and `transform` exclusively.
- **Bundle Footprint**: 0KB extra runtime for canvas noise (using base64 data-URI SVG pattern).

---

## 5. Responsiveness System & Device Test Matrix

### Responsive Breakpoints
```css
/* Mobile-first Media Queries */
--breakpoint-sm:  640px;  /* Large phones / phablets */
--breakpoint-md:  768px;  /* Tablets in portrait */
--breakpoint-lg:  1024px; /* Tablets landscape / Laptops */
--breakpoint-xl:  1280px; /* Standard desktop monitors */
--breakpoint-2xl: 1536px; /* Ultra-wide displays */
```

### Touch & Safe Area Guidelines
- **Touch Targets**: All interactive elements (buttons, links, form inputs) are guaranteed minimum **44px × 44px** hit area.
- **Safe Area Insets**:
  ```css
  padding-bottom: env(safe-area-inset-bottom, 16px);
  padding-top: env(safe-area-inset-top, 0px);
  ```
- **Mobile Navigation**: On screens `< 768px`, replace desktop horizontal menu with a bottom navigation bar or a clean slide-over drawer anchored to the right.

### Test Matrix
1. **Low-End Mobile**: Moto G54 / Galaxy A-series (Chrome, CPU throttled 4x, Slow 4G). Target: LCP < 2.5s, INP < 150ms.
2. **Flagship Mobile**: iPhone 15/16 Pro (Safari iOS 18, safe-area notches, dynamic island avoidance).
3. **Tablet**: iPad Pro 11" (Portrait & Landscape orientation shift tests).
4. **Desktop**: MacBook 14" / Windows 1440p (High-DPI retina display, mouse hover, focus ring navigation).

---

## 6. Mockup Wireframe Descriptions (Top 3 Pages)

### Page 1: Modern Marketing Showcase (Homepage)
```
┌────────────────────────────────────────────────────────────────────────┐
│ [Logo Gb-Tech]       [Solutions] [Work] [Pricing] [Company]   [Portal] │ (Glass Header)
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   [Badge: ✨ Engineering Scalable Web Applications]                   │
│   Architecting Intelligent Digital Products                            │
│   Engineered For Growth.                                               │
│                                                                        │
│   Gb-Technologies delivers high-impact web development, intuitive      │
│   interfaces, and resilient cloud architectures for modern companies.   │
│                                                                        │
│   [Primary CTA: Launch Your Project →]    [Secondary: Explore Case Studies]│
│                                                                        │
│   [Metrics Strip: 99.9% Uptime | 500+ Sprints | Sub-Second Latency]   │
│                                                                        │
├────────────────────────────────────────────────────────────────────────┤
│  SERVICES BENTO GRID (Col-Span-12 / Col-Span-8 / Col-Span-4)           │
│  ┌───────────────────────────────┐  ┌───────────────────────────────┐  │
│  │ 1. Full-Stack Web Development │  │ 2. Modern UI/UX Architecture  │  │
│  │    Interactive Code Preview   │  │    Figma-to-Code Pipeline     │  │
│  └───────────────────────────────┘  └───────────────────────────────┘  │
│  ┌───────────────────┐  ┌─────────────────────┐  ┌──────────────────┐  │
│  │ 3. Cloud & DevOps │  │ 4. Product Strategy │  │ 5. 24/7 Support  │  │
│  └───────────────────┘  └─────────────────────┘  └──────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

### Page 2: Service Detail & Interactive Project Estimator
```
┌────────────────────────────────────────────────────────────────────────┐
│ Breadcrumb: Solutions > Custom Web Applications                        │
│                                                                        │
│ <h1>Custom Full-Stack Web Applications</h1>                            │
│ <p>From high-throughput APIs to dynamic frontends, we engineer... </p> │
│                                                                        │
│ INTERACTIVE SCOPE ESTIMATOR (Client-Side State)                        │
│ ┌────────────────────────────────────────────────────────────────────┐ │
│ │ 1. Platform Type:  (•) Web App   ( ) E-Commerce   ( ) SaaS Portal  │ │
│ │ 2. Architecture:   (•) Decoupled API  ( ) Server-Rendered          │ │
│ │ 3. Key Modules:    [x] Auth & RBAC  [x] Payments  [ ] AI Search   │ │
│ │                                                                    │ │
│ │ Estimated Timeline: 4-6 Weeks  |  Deliverable: Production Deployment│
│ │ [Lock In Estimate & Book Consultation →]                           │ │
│ └────────────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────────┘
```

### Page 3: Authenticated Client Portal & Dashboard
```
┌────────────────────────────────────────────────────────────────────────┐
│ [Logo] Dashboard  Projects  Invoices  Settings          [Avatar: Gbure]│
├──────────────────────┬─────────────────────────────────────────────────┤
│ Active Project       │ Recent Milestones & Deliverables                │
│ ┌──────────────────┐ │ ┌─────────────────────────────────────────────┐ │
│ │ Gb-Tech Portal   │ │ │ Sprint 4: API Decoupling & Token Auth       │ │
│ │ Status: In Sprints│ │ │ Status: Completed  |  Verified by QA        │ │
│ │ Progress: [85%]  │ │ └─────────────────────────────────────────────┘ │
│ └──────────────────┘ │ ┌─────────────────────────────────────────────┐ │
│ Quick Actions        │ │ Sprint 5: Next.js Frontend Hydration        │ │
│ [Upload Asset]       │ │ Status: In Progress | Due: Tomorrow         │ │
│ [Request Change]     │ │ └─────────────────────────────────────────────┘ │
└──────────────────────┴─────────────────────────────────────────────────┘
```
