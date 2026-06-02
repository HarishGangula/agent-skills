# Colors and Tokens

## Overview

**Never hardcode color values** (`#3a3a3a`, `rgb(...)`, `hsl(...)`, named colors) directly in rules or inline. Define them once as variables/tokens and reference them everywhere. Hardcoded colors mean a rebrand or a dark-mode pass becomes a find-and-replace across the codebase, with no guarantee `#fff` in one file means the same thing as `#FFFFFF` in another.

If a token system already exists, **conform to it** — match its naming and layering. If none exists, **propose one** following W3C CSS Custom Properties conventions.

### Recommended token structure (when proposing one)

Two layers, per common W3C/design-system practice:

1. **Primitive (literal) tokens** — the raw palette, named by what they *are*:
   ```css
   :root {
     --blue-600: #2b6cb0;
     --gray-900: #1a202c;
     --gray-100: #f7fafc;
   }
   ```
2. **Semantic (role) tokens** — named by what they *do*, referencing primitives:
   ```css
   :root {
     --color-link: var(--blue-600);
     --color-text: var(--gray-900);
     --color-surface: var(--gray-100);
   }
   ```

Components reference **semantic** tokens, never primitives or raw hex. This makes theming (e.g. dark mode) a matter of remapping semantic tokens:
```css
[data-theme="dark"] {
  --color-text: var(--gray-100);
  --color-surface: var(--gray-900);
}
```

Naming: lowercase, hyphenated, prefixed by role (`--color-`, `--bg-`, `--border-`). Custom properties inherit and cascade — define the global set on `:root`.

### Framework note
- **Tailwind**: map colors to `tailwind.config.js` `theme.colors`; use `text-link` not `text-[#2b6cb0]`.
- **SCSS**: `$variables` or maps are acceptable, but prefer CSS custom properties when runtime theming (dark mode, multi-tenant) is needed, since SCSS vars are compiled away.

## When to Use

Apply whenever a literal color appears in a declaration: `color`, `background`, `border-color`, `fill`, `stroke`, `box-shadow`, `outline`, gradients.

## Common Rationalizations

- *"It's only used once."* Colors are never used once for long; the moment a second usage appears, you have drift.
- *"SCSS variables already handle this."* They handle authoring-time reuse but compile to literals — no runtime theming. Use custom properties for anything themed.
- *"The hex is self-documenting."* `#2b6cb0` tells you nothing about role; `--color-link` does.
- *"Dark mode is out of scope."* Tokenizing now costs little; retrofitting it across hardcoded hex later costs a lot.

## Red Flags

- Any `#hex`, `rgb()`, `hsl()`, or named color literal in a component rule.
- The same color expressed inconsistently (`#fff`, `#FFFFFF`, `white`, `rgb(255,255,255)`).
- Tailwind arbitrary color values: `bg-[#1a202c]`.
- Colors defined as SCSS vars but a dark-mode/theming requirement exists.

## Verification

- Grep for `#`, `rgb`, `hsl`, and common color names in declarations.
- Confirm each maps to a semantic token; if proposing a system, verify components reference semantic tokens, not primitives.
- Check contrast of the chosen tokens against WCAG AA (4.5:1 body text) — tokenizing is a good moment to fix contrast.
- After refactor, flip the theme (or temporarily remap a semantic token) and confirm the UI follows.

## Before / After

**Before**
```css
.btn-primary { background: #2b6cb0; color: #ffffff; }
.link { color: #2b6cb0; }
.alert { border: 1px solid #2b6cb0; }
```

**After**
```css
:root {
  --blue-600: #2b6cb0;
  --white: #ffffff;
  --color-brand: var(--blue-600);
  --color-on-brand: var(--white);
}

.btn-primary { background: var(--color-brand); color: var(--color-on-brand); }
.link { color: var(--color-brand); }
.alert { border: 1px solid var(--color-brand); }
```
One change to `--color-brand` now reskins all three.