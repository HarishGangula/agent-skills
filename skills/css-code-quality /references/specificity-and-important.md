# Specificity and !important

## Overview

**Don't reach for `!important` to win style conflicts.** It's an escape hatch overriding the normal cascade, and once one rule uses it, the only way to beat that rule is *another* `!important` — so it escalates into an arms race that makes the stylesheet progressively unmaintainable. Instead, **manage specificity deliberately** so the right rule wins by the cascade's normal rules.

### How specificity works (the lever to pull instead)
Specificity is counted as (inline, IDs, classes/attributes/pseudo-classes, elements). Higher wins; ties go to source order. To make a rule win you don't need `!important` — you need slightly higher (but still low) specificity, or later source order, or better structure:

- **Add a class** rather than fighting: `.card.is-active` beats `.card`.
- **Increase via a parent scope**: `.sidebar .link` beats `.link`.
- **Use source order / layer order**: later rules of equal specificity win.
- **Use `@layer`** (modern, ideal): cascade layers define override order explicitly without specificity inflation. Component layer beats base layer regardless of selector strength.
- **`:where()`** to *lower* specificity intentionally for easily-overridable base styles (`:where(.btn)` has zero specificity).

### Legitimate `!important` (document it)
- Overriding a third-party widget that ships `!important` and exposes no API.
- Angular Material / component-library internals reachable only via `::ng-deep` + `!important`.
- Single-purpose utility classes whose entire job is to win (`.u-hidden { display: none !important; }`) — acceptable as a deliberate, isolated convention, ideally confined to a utilities `@layer`.

The test: is `!important` here a *documented, contained override strategy*, or patching a specificity mistake? Former is fine; latter is the blocker.

## When to Use

Apply whenever you see `!important`, ID selectors used for styling, deeply chained selectors written purely to out-specify something, or `::ng-deep`/`>>>` deep-piercing combinators.

## Common Rationalizations

- *"It's the only thing that works."* It's the only thing tried. Inspect what rule is winning and out-specify it cleanly or reorder.
- *"It's quicker."* Quicker now; every future override of this property now also needs `!important`.
- *"The other rule is out of my control."* Sometimes true (third-party) — then it's a documented exception, scoped as tightly as possible, not a blanket habit.
- *"I'll use an ID, that's not !important."* IDs cause the same escalation one tier down; prefer classes and layers.

## Red Flags

- `!important` on ordinary component properties (`color`, `margin`, `display`) in first-party CSS.
- Multiple `!important`s competing on the same property (the arms race).
- ID selectors (`#header .nav`) used for styling rather than JS hooks.
- Long descending chains (`.a .b .c .d .e {}`) existing only to beat another rule.

## Verification

- Grep for `!important`. Classify each: documented third-party/utility override (keep, note it) vs. specificity patch (fix).
- For a patch, open devtools, find the winning rule, and beat it by adding a class, scoping to a parent, reordering, or moving into a cascade layer.
- Confirm removal didn't change rendering and the new winning rule has the *lowest* specificity that works.
- Prefer introducing `@layer` if the codebase has recurring override pain.

## Before / After

**Before**
```css
.button { background: gray; }
/* somewhere else, fighting the above */
.sidebar .button { background: blue !important; }
/* and now a new override also needs !important */
.sidebar .button.danger { background: red !important; }
```

**After (cascade layers — no !important)**
```css
@layer base, components, utilities;

@layer base {
  .button { background: var(--color-neutral); }
}
@layer components {
  .sidebar .button { background: var(--color-brand); }
  .sidebar .button.danger { background: var(--color-danger); }
}
```
The `components` layer wins over `base` by layer order, regardless of selector specificity — and `.danger` wins by source order within the layer. No `!important` anywhere.

**Acceptable, documented use**
```css
@layer utilities {
  /* Utility's whole job is to win; contained to the utilities layer. */
  .u-visually-hidden { position: absolute !important; clip-path: inset(50%) !important; }
}
```
