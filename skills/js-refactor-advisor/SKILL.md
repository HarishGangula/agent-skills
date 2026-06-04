---
name: js-refactor-advisor
description: Review JavaScript/TypeScript code and flag opportunities to simplify it — either by using modern native JS or by reaching for a well-chosen open-source library (lodash, dayjs/date-fns, zod/ajv, p-limit, ky, ts-pattern, immer, and more). Use this skill whenever the user asks to "review", "refactor", "simplify", "clean up", "modernize", or "find improvements in" JS/TS code, asks "what library should I use for this", asks whether some hand-rolled code can be replaced by a utility, or shares a snippet, a diff/PR, or points at a repo and wants suggestions. Trigger even when the user does not name a specific library — surface the right one. This skill ONLY flags and recommends; it does not rewrite code or produce diffs to apply.
license: MIT
metadata:
  author: Harish Kumar Gangula
  version: "2.0"
  last_updated: "2026-06-04"
---

# JS Refactor Advisor

Review JavaScript/TypeScript code and produce **advisory findings**: places where the code can be simplified, made more correct, or made lighter. This skill **flags and recommends only** — it never rewrites the user's code or outputs a diff for them to apply. Each finding shows a short before/after *illustration* so the recommendation is concrete, but the decision to apply it stays with the developer.

## Core philosophy: native first

The single most important behavior of this skill is restraint. **Adding a dependency is a cost** — bundle size, supply-chain surface, version churn, onboarding burden. A finding must justify that cost.

Apply this decision ladder to every candidate:

1. **Can modern native JS do it cleanly?** → recommend native. If the code *already* uses a library for something native now handles well, recommend *removing* that usage.
2. **Is native awkward, verbose, or error-prone here?** → recommend a focused library.
3. **Is the code already using a heavier library where a lighter/native option fits?** → recommend the swap.
4. **Does the repo already standardize on a library for this job?** → recommend using *that* one consistently. Never introduce a second library that does the same thing (e.g. don't add zod to an ajv codebase).

Modern native JS has eaten a lot of Lodash. Before recommending Lodash (or flagging its removal), consult `references/native-vs-lodash.md`.

## Severity tiers

Every finding is assigned exactly one tier. When output contains many findings (e.g. a repo scan), order them by tier so the developer fixes what matters first.

- **🔴 Correctness** — the current code can produce wrong results, crash, leak memory, or hit rate/resource limits. These are bugs waiting to happen, not style. Examples: `Promise.all` over a large `.map` with no concurrency cap, fragile manual date math across DST/timezones, validation gaps at a trust boundary, `JSON.parse(JSON.stringify())` deep-clone losing `Date`/`Map`/`undefined`.
- **🟡 Maintainability / Simplification** — the code works but is harder to read or change than it needs to be. Examples: hand-rolled `groupBy`/`debounce`, deeply nested spread updates, nested switch/if chains on a discriminant, verbose `Date` formatting.
- **🟢 Bundle / Dependency hygiene** — the code carries weight it doesn't need. Examples: full Lodash import for one function, moment.js (legacy/heavy), axios used only for trivial GETs, two libraries doing the same job.

## How to run on each scope

The finding logic is identical across scopes; only what you read changes.

- **Snippet / single file** — analyze directly.
- **Diff / PR** — focus on changed lines, but read enough surrounding context to judge correctly. Don't flag pre-existing issues outside the diff unless the change touches them.
- **Whole repo** — prioritize hot/entry files and shared utilities. **Skip** `node_modules`, build/dist output, vendored/third-party code, and generated files. If findings are numerous, summarize counts per tier and show the top findings rather than an exhaustive dump.

Before recommending a library swap, check the repo's `package.json` (and lockfile) — recommend libraries already present where possible, and note when a suggestion would add a new dependency.

## Output format

ALWAYS structure the report like this:

```
## Refactor findings

**Summary:** <N> findings — <X> 🔴 Correctness, <Y> 🟡 Maintainability, <Z> 🟢 Bundle

### 🔴 Correctness

#### 1. <one-line title> — `path/to/file.js:line`
**Now:** <what the code does today, briefly>
```js
// before (illustrative)
```
**Suggest:** <native approach, or library + why it clears the native-first bar>
```js
// after (illustrative)
```
**Note:** <adds dependency X / already in package.json / removes dependency Y — if relevant>

### 🟡 Maintainability / Simplification
...

### 🟢 Bundle / Dependency hygiene
...
```

Rules for the report:
- Keep before/after snippets short — they illustrate, they are not patches to apply.
- If you recommend *adding* a dependency, say so explicitly in **Note**. If you recommend *removing* one, say that too.
- If nothing is worth flagging, say so plainly rather than inventing low-value findings. Noise destroys the value of a tiered report.
- Don't recommend two libraries for the same job in one report.

## Library catalog

Each entry lists what to look for, what to recommend, and the tier it usually lands in. Details, edge cases, and the native-first cutoffs are in the reference files — read them when a category is in play rather than relying on memory:

- `references/native-vs-lodash.md` — the "native has caught up" table; consult before any Lodash recommendation.
- `references/validation.md` — the zod vs ajv vs valibot decision (read before recommending any validator).
- `references/catalog.md` — full per-library detail for dates, async, HTTP, control flow, immutability, and the smaller utilities.

Quick index of what triggers a finding:

| Category | Look for | Recommend | Usual tier |
|---|---|---|---|
| General utilities | hand-rolled `groupBy`/`keyBy`/`debounce`/`throttle`, deep merge loops, `JSON.parse(JSON.stringify())` clone | native first (see ref), else **lodash-es** / **es-toolkit** | 🟡 / 🔴 (clone) |
| Lodash bloat | full `import _ from 'lodash'`, or Lodash calls native now covers | tree-shaken import or native removal | 🟢 |
| Dates | manual `Date` arithmetic, hand-built formatting, **moment.js** | **dayjs** (small) or **date-fns** (tree-shake); flag moment as legacy | 🔴 (math) / 🟡 / 🟢 (moment) |
| Validation | hand-rolled `typeof`/shape checks, unvalidated boundary input | **zod** / **ajv** / **valibot** — see `references/validation.md` | 🔴 (boundary) / 🟡 |
| Async concurrency | `Promise.all` over `.map` with no cap, manual batching | **p-limit** / **p-map** | 🔴 |
| HTTP | axios for trivial requests, hand-rolled retry/timeout | native **fetch** or **ky** | 🟢 / 🟡 |
| Control flow | nested `switch`/`if-else` on a discriminant/shape | **ts-pattern** | 🟡 |
| Immutable updates | deeply nested spread updates `{...s,a:{...s.a,b:{...}}}` | **immer** | 🟡 |
| IDs | `Math.random().toString(36)` ad-hoc ids | **nanoid** (or `crypto.randomUUID()` native) | 🔴 (collisions) / 🟢 |
| className building | messy template-literal class concatenation (React) | **clsx** / **classnames** | 🟡 |
| Event emitting | hand-rolled emitter objects in browser | **mitt** / **nanoevents** | 🟡 |
| Global state (flag only) | tangled `useReducer` + context boilerplate | mention **zustand** / **valtio** as an option, don't push | 🟡 |

When a category comes up, read its reference section before writing the finding — the cutoffs (e.g. *when* native beats Lodash, *which* validator fits) are what make the recommendation trustworthy.