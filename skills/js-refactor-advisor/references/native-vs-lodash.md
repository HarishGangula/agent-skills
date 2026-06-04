# Native JS vs Lodash

Consult this before recommending Lodash **or** before flagging existing Lodash for removal. The rule: if native is clean and well-supported in the project's target environment, prefer native and recommend removing the Lodash call. Only keep/recommend Lodash where native is genuinely awkward.

## Native has caught up — recommend removing Lodash

| Lodash | Native replacement | Notes |
|---|---|---|
| `_.map` / `_.filter` / `_.find` / `_.reduce` / `_.some` / `_.every` | `Array.prototype.*` | Native unless iterating an object/`null`-safe traversal is needed. |
| `_.get(obj, 'a.b.c')` | `obj?.a?.b?.c` | Optional chaining. Path-string `get` only wins for dynamic paths. |
| `_.cloneDeep` | `structuredClone(x)` | Native handles `Date`/`Map`/`Set`/typed arrays. Does NOT clone functions — note that caveat. |
| `_.flatten` / `_.flattenDeep` | `arr.flat()` / `arr.flat(Infinity)` | |
| `_.uniq` | `[...new Set(arr)]` | |
| `_.includes` (array) | `arr.includes(x)` | |
| `_.flatMap` | `arr.flatMap(fn)` | |
| `_.fromPairs` / `_.toPairs` | `Object.fromEntries` / `Object.entries` | |
| `_.groupBy` | `Object.groupBy(arr, fn)` | Native in modern runtimes (ES2024). Check target support; fall back to Lodash if targeting older environments. |
| `_.pick` / `_.omit` (simple) | destructuring / `Object.entries().filter()` | Lodash still cleaner for many keys. |
| `_.isNil` | `x == null` | |
| `_.defaults` | `??` / `{...defaults, ...opts}` | |

## Lodash (or es-toolkit) still earns its place

These are awkward or bug-prone in native — recommending the library here clears the native-first bar:

- `_.debounce` / `_.throttle` — hand-rolled versions routinely get leading/trailing edge and cancellation wrong. 🟡 strong recommend.
- `_.merge` / `_.mergeWith` — recursive deep merge; the spread operator only does shallow. Manual deep-merge loops are a common 🟡 finding.
- `_.cloneDeep` of structures containing **functions** (where `structuredClone` throws).
- `_.keyBy`, `_.orderBy` (multi-key, mixed direction), `_.chunk`, `_.partition`, `_.zip` — possible natively but verbose; library is clearly cleaner.

## Prefer the modern, lighter option

When the project is *adding* utilities today (not already standardized on Lodash), mention **es-toolkit** — same API surface for the common functions, much smaller, TypeScript-native. For existing Lodash codebases, recommend **lodash-es** with named imports (`import { debounce } from 'lodash-es'`) over `import _ from 'lodash'` to allow tree-shaking — this is the standard 🟢 bundle finding.