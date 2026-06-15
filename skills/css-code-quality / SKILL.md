---
name: css-code-quality
description: Review, refactor, and optimize CSS/SCSS for best practices. Use this skill whenever the user wants to review CSS, refactor stylesheets, clean up styles, enforce CSS conventions, or asks about CSS best practices, code smells, or "is this CSS good". Trigger on phrases like "review my CSS", "refactor this stylesheet", "clean up these styles", "check my CSS", "optimize this CSS", "why isn't this style applying", or when a .css/.scss file or a block of CSS is shared for feedback — even if the word "review" isn't used. Also trigger when the user shares inline styles, hardcoded colors, px-heavy sizing, or `!important` and asks whether it's okay. Covers plain CSS, SCSS, and framework contexts (Tailwind, Bootstrap, Angular Material).
license: MIT
metadata:
  author: Harish Kumar Gangula
  version: "1.0"
  last_updated: "2026-06-03"
---

# CSS Code Quality

Review, refactor, optimize CSS against consistent best practices grounded in W3C/MDN guidance. Three separable things — user can ask for any one:

1. **Review** — read CSS, report findings (no edits), tiered by severity.
2. **Refactor** — rewrite CSS to fix findings, preserving visual output.
3. **Optimize** — reduce redundancy, collapse selectors, remove dead rules.

Default to **review first**, then offer refactor. Jump straight to refactoring only if user explicitly asks to "fix/refactor/rewrite".

## Workflow

1. **Detect context first.** Before flagging, determine framework and conventions in play — rules bend on it. Look for:
   - A framework: Tailwind (utility classes like `flex gap-4`), Bootstrap (`btn btn-primary`, `.row`/`.col`), Angular Material (`mat-*`, `::ng-deep`), or plain CSS/SCSS.
   - An existing token system: `:root { --color-... }`, SCSS `$variables`, or a theme file. If one exists, conform to its naming. If none, propose one per W3C custom-property conventions (see `references/colors-and-tokens.md`).
   - Build constraints: SCSS nesting, CSS Modules, scoped styles (Vue/Angular), CSS-in-JS.
2. **Apply the five concern areas.** Each has a dedicated reference file with rule, common rationalizations, red flags, verification steps, before/after examples. Read the relevant reference when a concern is in scope:
   - `references/inline-styles.md` — no inline CSS unless the framework enforces it
   - `references/sizing-units.md` — `rem` for spacing/typography-scaled values, with documented exceptions
   - `references/colors-and-tokens.md` — no hardcoded colors; use variables/tokens
   - `references/specificity-and-important.md` — no `!important`; manage specificity instead
   - `references/additional-best-practices.md` — magic numbers, logical properties, nesting depth, `gap`, `clamp()`
3. **Report findings tiered by severity** (see Output format).
4. **Offer the refactor.** If reviewing, end by offering the corrected CSS.

## Severity tiers

Classify every finding so the review is actionable, not a flat list:

- **Blocker** — breaks maintainability or theming at scale, or indicates a real bug (e.g. `!important` used to win a specificity war, hardcoded brand color duplicated across files, inline style in a non-enforcing context).
- **Warning** — works today but bites later (e.g. `px` for component padding, magic numbers, deep nesting `> 3` levels).
- **Nit** — stylistic or minor (e.g. shorthand could collapse, property ordering).

Never invent blockers to pad a review. If the CSS is clean, say so.

## Framework awareness (important)

Rules are not absolute — they yield to the framework's own conventions:

- **Tailwind**: Utility classes ARE the convention; do not flag `class="px-4 text-red-500"` as "inline-like" or "hardcoded color" — idiomatic. Flag arbitrary values (`text-[#3a3a3a]`, `mt-[13px]`) where a theme token exists, and flag real inline `style="..."` attributes. Colors should map to `tailwind.config` theme tokens.
- **Bootstrap**: Prefer utility/component classes over custom CSS; if overriding, use SCSS variable overrides (`$primary`) not `!important`.
- **Angular Material**: `::ng-deep` and some `!important` are sometimes the only way to override component internals — treat as a documented, scoped exception, not a blocker, but note it and suggest the theming API where one exists.
- **Plain CSS/SCSS**: All rules apply at full strength.

When unsure which framework is in play, ask before flagging framework-idiomatic patterns as errors.

## Output format

ALWAYS use this structure for a review:

```
## CSS Review

**Context:** [framework detected, token system found/absent, build setup]

### Blockers
- [file:line or selector] — [issue] → [fix, one line]

### Warnings
- ...

### Nits
- ...

### Summary
[1-2 sentences: overall health + what to prioritize]
```

Per-finding rationale and before/after examples live in the reference files — pull concrete before/after snippets into the report when they make the fix clearer.

For a **refactor**, output the corrected CSS (as a file if substantial, inline if short), then a short changelog mapping each change back to its concern area so the user sees what moved and why.

## Reference files

- `references/inline-styles.md`
- `references/sizing-units.md`
- `references/colors-and-tokens.md`
- `references/specificity-and-important.md`
- `references/additional-best-practices.md`

Read the ones relevant to the CSS in front of you, not all five every time.
