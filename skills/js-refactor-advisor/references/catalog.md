# Library catalog — detail

Per-category detail and before/after illustrations. Read the relevant section when that category appears in the code under review.

## Dates

**dayjs** — tiny (~2KB), moment-like API, immutable. Default recommendation for general date handling.
**date-fns** — pure functions, fully tree-shakeable; better when you use only a few functions or want to ship the minimum.
**moment.js** — 🟢 flag as legacy: large, mutable, in maintenance mode. Recommend migrating to dayjs (near drop-in) or date-fns.

Findings:
- 🔴 **Manual date math** — adding/subtracting via `new Date(d.getTime() + 86400000)` breaks across DST. Recommend `dayjs(d).add(1, 'day')` or date-fns `addDays`.
- 🟡 **Hand-built formatting** — `${d.getFullYear()}-${String(d.getMonth()+1).padStart(2,'0')}-...`. Recommend `dayjs(d).format('YYYY-MM-DD')`. (Note: `Intl.DateTimeFormat` is the native option for locale-aware display — mention it when formatting is for display only.)

## Async concurrency

**p-limit** / **p-map** — cap concurrency on async work.

- 🔴 **Unbounded `Promise.all`** — `await Promise.all(items.map(fetchThing))` over a large array fires every request at once: rate-limit hits, memory spikes, connection exhaustion. Recommend `pMap(items, fetchThing, { concurrency: 5 })` or a `p-limit` gate. This is one of the highest-value findings the skill produces.
- 🟡 **Manual batching loops** — hand-written chunk-and-await loops to limit concurrency reimplement p-map poorly.

## HTTP

**native fetch** — available everywhere modern (Node 18+, all browsers).
**ky** — tiny wrapper over fetch adding retries, timeouts, hooks, JSON by default.

- 🟢 **axios for trivial requests** — if axios is used only for simple GET/POST with JSON, native fetch (or ky) removes the dependency. Don't push this if axios is used for interceptors, progress, or wide browser support needs.
- 🟡 **Hand-rolled retry/timeout wrappers** around fetch — recommend ky, which has these built in.

## Control flow

**ts-pattern** — exhaustive, type-safe pattern matching.

- 🟡 **Nested switch/if-else on a discriminant** — matching on `action.type`, a status enum, or a tagged-union shape with deep nesting. ts-pattern flattens it and (in TS) enforces exhaustiveness so a new case can't be silently unhandled.

## Immutable updates

**immer** — write "mutating" code that produces an immutable next state.

- 🟡 **Deeply nested spread updates** — `{...s, a: {...s.a, b: {...s.a.b, c: v}}}`. Error-prone and unreadable past one level. Recommend immer's `produce(s, draft => { draft.a.b.c = v })`. Especially relevant in reducers.

## Smaller utilities

- **nanoid** — 🔴 if `Math.random().toString(36).slice(2)` is used for IDs that need uniqueness (collision risk); 🟢 if it's a non-critical throwaway. Note `crypto.randomUUID()` is the native option when a UUID is acceptable.
- **clsx** / **classnames** — 🟡 messy conditional className building via template literals / array joins in React. `clsx('base', { active: isActive })` is clearer. clsx is the smaller of the two.
- **mitt** / **nanoevents** — 🟡 hand-rolled event-emitter objects (`{ listeners: [], on(){}, emit(){} }`) in browser code. Tiny, tested replacements.
- **zustand** / **valtio** — flag only, don't push. When `useReducer` + context boilerplate for global state has grown tangled, *mention* these as lighter-than-Redux options, leaving the call to the developer.

## es-toolkit

Modern, much smaller Lodash alternative with TS-native types. Mention when the project is adding utilities today rather than already standardized on Lodash. For existing Lodash users, the lower-friction 🟢 finding is switching `import _ from 'lodash'` to named `lodash-es` imports for tree-shaking.