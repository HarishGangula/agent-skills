# Sizing Units

## Overview

Use **`rem` for spacing- and typography-scaled values**: `font-size`, `padding`, `margin`, `gap`, `width`/`height` of text-driven components, `border-radius`. `rem` is relative to the root font size, so the whole layout scales with the user's browser font preference — a real accessibility win — and it gives one consistent scale instead of arbitrary pixel values scattered everywhere.

It is **not** "always rem." Some properties are genuinely better in other units, and forcing `rem` there is wrong:

| Use | Unit | Why |
|-----|------|-----|
| Hairlines / borders | `px` | A 1px border should stay 1px; `rem` makes it scale oddly and can sub-pixel-blur. |
| Full-viewport sizing | `vh` / `vw` / `dvh` / `svh` | Hero sections, full-height panels. `dvh`/`svh` handle mobile browser chrome. |
| Proportional within a container | `%` | Fluid columns, images filling a parent. |
| Element-relative spacing tied to its own font | `em` | Padding inside a button that should grow with *that* button's text. |
| Fluid/responsive scaling | `clamp(min, preferred, max)` | Replaces media-query step-ups for type and spacing. |

The principle: `rem` for the shared spacing/type scale; the other units where the value is intrinsically relative to something other than the root.

## When to Use

Apply whenever you see numeric length values on `font-size`, `padding`, `margin`, `gap`, `width`, `height`, `min/max-*`, `border-radius`, `inset`, or `top/right/bottom/left`.

## Common Rationalizations

- *"px is more precise / predictable."* Predictable until a user bumps their default font size and your `px` layout ignores them entirely.
- *"Converting everything to rem is tedious."* It's a one-time scale definition; after that you reuse tokens, not raw numbers.
- *"Designers gave me px in Figma."* Translate to the rem scale (px ÷ 16 = rem at default root). The handoff value isn't the implementation contract.
- *"rem on borders is fine."* It technically works but defeats the purpose and risks blur — borders/hairlines are a documented px exception.

## Red Flags

- `font-size: 14px`, `padding: 12px 20px`, `margin-bottom: 24px` — the spacing/type scale in raw px.
- A pile of unrelated pixel values (`13px`, `17px`, `23px`) with no scale → magic numbers (see additional-best-practices).
- `height: 100vh` on a mobile layout (use `dvh`/`svh`).
- `width: 300px` on a container that should be fluid (likely `%`, `max-inline-size`, or `clamp()`).

## Verification

- Grep for `px` and triage each: is it a border/hairline (keep), a viewport/proportional case (different unit), or spacing/type (→ rem)?
- Confirm a root font size baseline (`html { font-size: 100%; }` = 16px) so `rem` math is predictable. Avoid setting `html { font-size: 62.5% }` tricks unless the codebase already relies on it.
- After conversion, zoom the browser / change the OS font size and confirm the layout scales rather than breaking.

## Before / After

**Before**
```css
.card {
  width: 320px;
  padding: 16px 24px;
  border: 1px solid #ddd;
  border-radius: 8px;
  font-size: 14px;
  height: 100vh;
}
```

**After**
```css
.card {
  inline-size: 20rem;          /* 320 / 16 — scales with root */
  padding: 1rem 1.5rem;        /* spacing scale → rem */
  border: 1px solid var(--color-border); /* hairline stays px */
  border-radius: 0.5rem;       /* radius → rem */
  font-size: 0.875rem;         /* type → rem */
  min-block-size: 100dvh;      /* viewport height, mobile-safe */
}
```