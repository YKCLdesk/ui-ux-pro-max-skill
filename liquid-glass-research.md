# iOS 26 Liquid Glass - Verified Research

**Project:** Fuse Main System (Enterprise CRM)
**Stack:** React 19 + Next.js 15 + MUI 7.2.0 + Tailwind 4.1.4 + Framer Motion
**Mode:** Light mode primary (bg: #F8FAFC, paper: #FFFFFF)
**Compiled:** 2026-02-15 | Verified by Opus 4.6

---

## What iOS 26 Liquid Glass Actually Is

Introduced WWDC 2025. NOT just backdrop-filter blur (that's 2020 glassmorphism).

Real liquid glass requires:
- Edge refraction (light bending on curved surfaces)
- SVG displacement mapping (localized distortion)
- Chromatic aberration (subtle RGB split on edges)
- Specular highlights (dynamic light response)
- Fluid morphing animations (400-600ms organic curves)
- Multi-layer depth with z-index hierarchy

---

## Verified NPM Packages (Real, on npmjs.com)

### 1. liquid-glass-react (PRIMARY)
- **npm:** `npm install liquid-glass-react`
- **Version:** 1.1.1 | **GitHub Stars:** 4,500+ | **License:** MIT
- **Author:** rdev (github.com/rdev/liquid-glass-react)
- **Verified Props:**
  - `displacementScale` (default: 70) - refraction intensity
  - `blurAmount` (default: 0.0625) - frosting level
  - `saturation` (default: 140) - color intensity
  - `elasticity` (default: 0.15) - animation fluidity
  - `cornerRadius` (default: 999) - border rounding
  - `mode` - standard | polar | prominent | shader
  - `mouseContainer` - ref for mouse tracking area
- **CRITICAL LIMITATION:** Displacement/refraction NOT visible on Safari or Firefox. Only Chromium (Chrome/Edge) gets full effect. Falls back to blur-only on others.
- **Demo:** https://liquid-glass.maxrovensky.com

### 2. @nkzw/liquid-glass (FORK - fixes)
- **npm:** `npm install @nkzw/liquid-glass`
- **Version:** 1.3.0 | Fork of liquid-glass-react with styling + TypeScript fixes
- **Same API** as liquid-glass-react but better maintained

### 3. @liquidglass/react (SMALL)
- **npm:** `npm install @liquidglass/react`
- **Version:** 0.1.3 | **Downloads:** ~1,457 | 48.3 kB
- **Uses:** Backdrop filters + SVG displacement mapping
- **Status:** Early/small - not battle-tested

### 4. @specy/liquid-glass-react (THREE.JS)
- **npm:** `npm install @specy/liquid-glass-react`
- **HEAVY:** 6.83 MB bundle (Three.js dependency)
- **Best effect quality** but inappropriate for production CRM due to bundle size

### DOES NOT EXIST:
- ~~liquid-glass-ui~~ - No such npm package. Website liquidglassui.org exists but no installable package.

---

## CSS-Only Approach (No Library)

For cross-browser consistency, CSS-only glass works everywhere:

```css
/* Light Mode Liquid Glass */
.glass-card {
  background: rgba(255, 255, 255, 0.72);
  backdrop-filter: blur(16px) saturate(180%);
  -webkit-backdrop-filter: blur(16px) saturate(180%);
  border: 1px solid rgba(255, 255, 255, 0.45);
  border-radius: 16px;
  box-shadow:
    0 4px 24px rgba(30, 58, 95, 0.08),
    inset 0 1px 0 rgba(255, 255, 255, 0.6);
}

/* Needs a colored/gradient background BEHIND the glass to show the effect */
.glass-backdrop {
  background: linear-gradient(135deg, #1e3a5f 0%, #3b82f6 50%, #0ea5e9 100%);
}
```

**Browser support for backdrop-filter:** 96%+ (works everywhere including Safari/Firefox).

---

## Light Mode Considerations (CRITICAL for this project)

Current app background is #F8FAFC (near-white). Glass on near-white = invisible effect.

**Solutions:**
1. Add vibrant gradient backgrounds behind glass panels (sidebar, headers, hero sections)
2. Use higher opacity for glass in light mode: rgba(255,255,255, 0.6-0.8) not 0.1-0.3
3. Rely on border + shadow + subtle blur rather than transparency alone
4. Use colored tint glass: rgba(30,58,95, 0.04) for primary-tinted panels
5. Text must stay #1E293B (current primary) - contrast ratio 4.5:1 minimum

**What already works in this codebase:**
- StyledPageLayout header: `backdrop-filter: blur(10px)` with gradient background
- This pattern is the right approach - gradient backdrop + glass overlay

---

## RTL / Hebrew - Critical Requirements

This is an RTL-first Hebrew application. `dir="rtl"` is hardcoded in layout.tsx.
Default direction: `'rtl'` in FuseDefaultSettings. `stylis-plugin-rtl` is installed for MUI emotion RTL.
All UI must work flawlessly in RTL.

### Liquid Glass RTL Rules

- **Gradients:** Flip direction. `135deg` in LTR becomes `225deg` in RTL.
  Use `[dir="rtl"]` selector override or CSS logical keywords.
- **Light source:** iOS liquid glass assumes top-left light. In RTL, mirror to top-right.
  For `inset` box-shadows simulating highlights, flip the x-offset in RTL.
- **Animations:** Horizontal motion must reverse. Framer Motion pattern:
  `x: isRTL ? 20 : -20` for directional slide animations.
- **SVG filters:** liquid-glass-react SVG displacement is direction-agnostic. No RTL issue.
- **Mouse tracking:** liquid-glass-react uses absolute coordinates. Works in RTL unchanged.
- **Tailwind RTL:** Codebase has `[dir="rtl"] [class*="space-x-"]` fix in index.css.
  Prefer `gap-*` over `space-x-*`. Use `ms-*`/`me-*` not `ml-*`/`mr-*` in new glass code.
- **MUI RTL:** MUI 7 auto-handles RTL via theme direction + `stylis-plugin-rtl`.
- **Text:** Hebrew is right-aligned by default. Never force `text-left` on glass cards.
- **Scrollbars:** Already handled in app-base.css for RTL positioning.
- **Icons in glass navbars:** Icons preceding text appear on the RIGHT side in RTL.

### RTL Glass Testing Checklist

1. Test every glass component with Hebrew text (varied word lengths)
2. Verify gradient directions render correctly in RTL
3. Check glass card shadows don't create asymmetry issues
4. Test glass overlays on both left and right sidebars
5. Verify glass modals center correctly in RTL
6. Confirm animations reverse direction properly

---

## Codebase Integration Rules

### MUST follow existing patterns

- Colors: Single source in `src/theme/colors.ts` and CSS vars in `src/styles/index.css`
- Styling: MUI `styled()` with `theme.vars.palette` for themed components
- Utilities: Tailwind classes via `className` for layout/spacing
- Animation: Framer Motion imported from `'motion/react'`
- Layout: FusePageSimple (header + sidebars + content)
- RTL: `dir="rtl"` is global default. Use logical properties. `stylis-plugin-rtl` installed.
- Components: PascalCase, styled roots suffixed with `Root`

### DO NOT

- Hardcode color values - use CSS vars or theme
- Skip `-webkit-backdrop-filter` (Safari needs it)
- Use glass on light backgrounds without a vibrant backdrop behind it
- Stack more than 2-3 glass layers (performance + readability)
- Forget `prefers-reduced-motion` for animations
- Break existing MUI component APIs when wrapping with glass
- Use `ml-`/`mr-`/`pl-`/`pr-` in new code - use `ms-`/`me-`/`ps-`/`pe-` for RTL
- Use physical gradient angles without RTL override
- Assume LTR light source direction

---

## Tools for Design Generation

### v0 by Vercel (https://v0.app/)
- Generates Next.js + Tailwind glassmorphism components
- Good for prototyping individual components
- Free tier available

### Glass UI Generator (https://ui.glass/generator/)
- Interactive CSS generator for glass effects
- Exports production-ready backdrop-filter CSS

### UI UX Pro Max Skill (installed)
- Search: `python3 scripts/search.py "liquid glass" --domain style`
- Design system: `python3 scripts/search.py "CRM dashboard" --design-system`
- Has both "Glassmorphism" (style #3) and "Liquid Glass" (style #14) in database

---

## Recommended Strategy

### Phase 1: CSS-first glass utilities
Add Tailwind utilities and MUI theme variants for glass effects.
No external library needed. Works on ALL browsers.

### Phase 2: liquid-glass-react for hero/showcase elements
Use `liquid-glass-react` or `@nkzw/liquid-glass` for specific premium components
where the refraction effect adds value (landing pages, dashboards, modals).
Accept that Safari/Firefox see a simpler fallback.

### Phase 3: Component-by-component migration
Convert existing MUI Paper/Card components to glass variants.
Start with navigation, then cards, then modals.
Always test light mode + RTL + mobile.
