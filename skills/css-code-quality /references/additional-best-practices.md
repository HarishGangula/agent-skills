# Additional Best Practices

Suggested additions beyond the four core rules. Same spirit: maintainability, scalability, accessibility. Apply with judgment — most are Warnings or Nits, not Blockers.

## Overview

A grab-bag of high-value CSS hygiene checks:

1. **Magic numbers** — unexplained literal values (`top: 37px`, `z-index: 9999`) with no derivation. They signal a value tuned by trial-and-error that no one dares touch. Replace with tokens, a documented scale, or a comment explaining the derivation.
2. **Logical properties** — prefer `margin-inline`, `padding-block`, `inset`, `inline-size`/`block-size` over physical `margin-left`, `padding-top`, `width`/`height`. They adapt automatically to RTL/vertical writing modes — important for i18n.
3. **Selector nesting depth** — keep specificity chains shallow (≤ 3 levels). Deep nesting (`.a .b .c .d .e`) inflates specificity (feeding the `!important` problem) and couples styles tightly to DOM structure.
4. **`gap` over margins** — in flex/grid, use `gap` for spacing between items instead of margins on children (no last-child margin hacks, no collapsing surprises).
5. **`clamp()` for fluid sizing** — replace stepped media queries for type/spacing with `clamp(min, preferred, max)` for smooth responsive scaling.

## When to Use

Scan for these alongside the four core rules during any review or refactor. They commonly co-occur: magic numbers often hide in `px` values; deep nesting often pairs with `!important`.

## Common Rationalizations

- *"The magic number works, don't touch it."* It works until layout around it shifts; an undocumented number is a latent bug.
- *"Logical properties are unfamiliar."* They're broadly supported now and free you from RTL rewrites later.
- *"Nesting mirrors my HTML, that's clearer."* It couples CSS to markup; a small DOM change silently breaks styles.
- *"Margins for spacing are fine."* They are, until you hit collapsing margins or last-child hacks; `gap` is cleaner where the layout is flex/grid.

## Red Flags

- `z-index: 9999` / `z-index: 99999` — z-index arms race; use a documented scale (`--z-modal: 1000`).
- Oddly specific offsets: `top: 37px`, `margin-left: -13px`.
- `width`/`margin-left`/`padding-right` everywhere in an app that needs RTL.
- `.nav ul li a span {}` — over-nested, structure-coupled.
- `> * + * { margin-top: ... }` owl-selector hacks where `gap` would do.
- Stacks of `@media` breakpoints adjusting one `font-size` step by step → `clamp()`.

## Verification

- Grep for `z-index`, large/odd `px` values, and physical-direction properties.
- Check max nesting depth in SCSS (lint rule: `max-nesting-depth: 3`).
- Confirm flex/grid containers use `gap`; check for last-child margin resets that `gap` would remove.
- For fluid type, verify `clamp()` bounds are sensible (min readable, max not enormous) and test at narrow + wide viewports.

## Before / After

**Magic number / z-index**
```css
/* before */ .modal { z-index: 9999; top: 37px; }
/* after  */
:root { --z-modal: 1000; }
.modal { z-index: var(--z-modal); top: var(--space-9); }
```

**Logical properties**
```css
/* before */ .card { margin-left: 1rem; padding-top: 0.5rem; width: 20rem; }
/* after  */ .card { margin-inline-start: 1rem; padding-block-start: 0.5rem; inline-size: 20rem; }
```

**gap over margins**
```css
/* before */
.list { display: flex; }
.list > * + * { margin-left: 1rem; }
/* after  */
.list { display: flex; gap: 1rem; }
```

**clamp() over breakpoints**
```css
/* before */
.title { font-size: 1.5rem; }
@media (min-width: 48rem) { .title { font-size: 2rem; } }
@media (min-width: 64rem) { .title { font-size: 2.5rem; } }
/* after  */
.title { font-size: clamp(1.5rem, 1rem + 2.5vw, 2.5rem); }
```
