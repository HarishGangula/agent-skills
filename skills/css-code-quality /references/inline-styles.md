# Inline Styles

## Overview

CSS belongs in stylesheets (or scoped style blocks), not in `style="..."` attributes on elements. Inline styles carry the highest specificity (short of `!important`), so they're nearly impossible to override cleanly later; they can't be reused, cached, or themed; and they scatter presentation across markup where no one looks for it. The rule: **no inline CSS unless the underlying framework enforces it.**

The "unless the framework enforces it" carve-out matters. Some setups legitimately produce style attributes:
- A charting/animation library writing computed `transform`/`width` at runtime.
- Dynamically computed values that genuinely depend on data (e.g. `style="--progress: 73%"` feeding a CSS variable — this is the *good* pattern, passing data in via a custom property rather than hardcoding the final look).
- Email HTML, where inline styles are the only thing that reliably renders.

Note that Tailwind/Bootstrap utility classes are NOT inline styles — they're classes, and they're idiomatic. Don't flag them here.

## When to Use

Apply this check whenever you see a `style="..."` attribute in HTML/JSX/templates, or a `[style.x]`/`:style` binding in Angular/Vue, or `style={{ }}` objects in React that contain static presentational values.

## Common Rationalizations

- *"It's just one quick override."* One-offs accumulate; the next dev copies the pattern. A utility class or a scoped rule is barely more effort.
- *"It's faster to write."* Faster to write, far slower to maintain and to override.
- *"The value is dynamic so it has to be inline."* Only the *data* is dynamic. Pass the data as a CSS custom property (`style="--w: 60%"`) and keep the actual styling in the stylesheet (`width: var(--w)`).
- *"It's React, inline is normal."* Static styling still belongs in CSS modules / styled rules / Tailwind. Reserve the `style` prop for values computed at render time.

## Red Flags

- `style="color: #fff; padding: 10px"` — static presentation in markup.
- Repeated identical inline styles across multiple elements (begging to be a class).
- Inline styles used specifically to beat an existing rule's specificity.
- React `style={{ color: 'red', fontSize: 14 }}` with literal, non-computed values.

## Verification

- Grep for `style=` / `:style` / `[style` / `style={{` in templates and components.
- For each hit, ask: is the value computed at runtime from data? If no → move to a class. If yes → is it passing data via a custom property, or hardcoding the final look? Prefer the custom-property pattern.
- After refactor, confirm the element renders identically and that the new rule's specificity is low enough to be overridable.

## Before / After

**Before**
```html
<div style="margin-top: 16px; color: #2b6cb0; font-weight: 600;">Title</div>
```

**After**
```html
<div class="section-title">Title</div>
```
```css
.section-title {
  margin-top: 1rem;
  color: var(--color-link);
  font-weight: 600;
}
```

**Dynamic value — the right way**
```html
<!-- data is dynamic, styling is not -->
<div class="progress-bar" style="--progress: 73%;"></div>
```
```css
.progress-bar { inline-size: var(--progress); }
```