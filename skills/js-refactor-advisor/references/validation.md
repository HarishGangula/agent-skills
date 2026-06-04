# Runtime validation: zod vs ajv vs valibot

Read this before recommending any validator. The libraries overlap but fit different contexts — recommend by context, not by personal default. **Never recommend introducing a second validator into a codebase that already standardized on one** — instead, the finding becomes "use the existing validator consistently here."

## Decision table

| Situation | Recommend | Why |
|---|---|---|
| TS codebase, want the static type inferred from the validator | **zod** | `z.infer<typeof schema>` keeps runtime check and TS type from drifting; one source of truth. |
| Validating against an existing/shared **JSON Schema**, or using Fastify/OpenAPI/AsyncAPI, or a hot request path | **ajv** | The schema is a language-agnostic shared contract; ajv compiles schemas to functions and is the fastest option for high-volume validation. |
| Plain JS (no TS), already has JSON Schemas | **ajv** | zod's main draw is TS inference — moot without TS. |
| TS, but bundle size is the hard constraint | **valibot** | Same inferred-type benefit as zod, modular and much smaller. |

## When to flag at all

- **🔴 Correctness** — unvalidated input crossing a trust boundary: HTTP request bodies, query params, env vars, third-party API responses, message-queue payloads, `JSON.parse` of external data. Hand-rolled `if (typeof x !== 'string')` chains that are incomplete or easy to drift from the real shape.
- **🟡 Maintainability** — validation logic that works but is sprawling, duplicated, or hand-maintained where a schema would be tighter.

## Notes for the finding

- ajv + zod solve overlapping problems; if the repo's request shapes are already expressed as JSON Schema (e.g. an OpenAPI/Sunbird-style envelope spec), ajv is the natural fit and lets the schema stay the shared contract across services.
- zod shines for in-app boundaries (forms, env parsing, internal function inputs) and for ergonomic `.transform()` / `.refine()` that are clumsy in raw JSON Schema.
- If recommending ajv in a TS project, mention `json-schema-to-ts` or ajv's typing helpers so types aren't lost.