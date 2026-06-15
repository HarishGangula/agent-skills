---
name: sunbird-api-spec
description: Use this skill whenever the user wants to design or document REST APIs in the Sunbird convention — including drafting API specifications for a new requirement, defining endpoint contracts between frontend and backend or between microservices, formalizing request/response envelopes, enumerating error responses, or producing Postman-style API documentation. Trigger on phrases like "design APIs for", "create an API spec", "document this endpoint", "API contract for", "Sunbird-style API", "turn this requirement into APIs", or any request to formalize a new feature into a set of endpoints. Use this even when the user does not say "Sunbird" explicitly, as long as they want REST APIs following a standard envelope pattern. Do NOT use for GraphQL, gRPC, or REST conventions that explicitly diverge from the Sunbird envelope.
license: MIT
metadata:
  author: Harish Kumar Gangula
  version: "2.0"
  last_updated: "2026-05-28"
---

# Sunbird API Spec

## Overview

Design and document RESTful APIs following Sunbird convention — one URL pattern, one request/response envelope, predictable error formats, consistent naming. Output is either a design proposal or finished API doc page a backend engineer can implement and a frontend engineer can consume without ambiguity.

## When to Use

- Designing new API endpoints for a feature or requirement
- Producing formal API docs (Postman-style reference or markdown spec)
- Revising existing APIs into Sunbird convention
- Reviewing an API implementation for compliance with this standard
- Defining contracts between frontend, backend, or microservices

## Workflow

Follow in order. Do not skip clarification — guessing on resource names or auth model leads to rework.

### 1. Clarify the requirement

Before writing spec, confirm with user:

- **Resource(s)** — domain entity (entities) the APIs manage. Use a single lowercase noun (e.g., `order`, `assessment`, `enrolment`).
- **Actions needed** — which standard verbs apply (`create`, `read`, `update`, `delete`, `list`, `search`) and any domain-specific verbs (`publish`, `retire`, `flag`, etc.).
- **Auth model** — same for every endpoint, or varies per action? Role-based restrictions to document?
- **Tenant or channel scoping** — do any endpoints need `X-Channel-Id`, `X-Tenant-Id`, or similar headers?
- **ID format** — UUIDs, prefixed IDs (`ord_...`), or other?
- **Output format** — markdown spec, Postman-style reference, or inline JSON examples? Default to markdown spec saved as a file unless user says otherwise.

If ambiguous on any above, ask before generating endpoints.

### 2. Enumerate endpoints first

Produce one table listing every endpoint *before* going deep on any one. Lets user correct the shape before details are filled in.

| # | Method | URL | API ID | Purpose |
|---|---|---|---|---|
| 1 | POST | `/orders/v1/create` | `api.order.create` | Create a new order |
| 2 | GET | `/orders/v1/read/{orderId}` | `api.order.read` | Fetch an order by ID |
| 3 | POST | `/orders/v1/search` | `api.order.search` | Search orders with filters |
| 4 | PATCH | `/orders/v1/update/{orderId}` | `api.order.update` | Update an order |
| 5 | DELETE | `/orders/v1/delete/{orderId}` | `api.order.delete` | Delete an order |

Wait for confirmation or corrections, then proceed.

### 3. Document each endpoint

For every endpoint in the list, produce the full endpoint block — see **Endpoint Documentation Template** below for exact structure.

### 4. Add cross-cutting sections

After endpoints, append:

- A **headers reference** section if any non-default headers are used
- An **error codes reference** table consolidating every `err` code used across endpoints
- A **changelog** stub for future versions

### 5. Save and present

Write document to `/mnt/user-data/outputs/<feature>-api-spec.md` and call `present_files` so user can download it. For very small specs (one or two endpoints), inline markdown is acceptable.

---

## API Conventions

### URL Pattern

```
/{resource}/{version}/{verb}[/{subVerb}]/{resourceId}
```

| Segment | Purpose | Example |
|---|---|---|
| **Resource** | The domain entity — **plural** in the URL (see rule 9) | `orders`, `users` |
| **Version** | API version | `v1`, `v2` |
| **Verb** | Action being performed | `create`, `read`, `publish` |
| **Sub-verb** | Optional, for compound actions only | `flag/accept`, `flag/reject` |
| **Identifier** | Optional, the specific resource instance — always last | `{orderId}` |

### Verb to HTTP Method Mapping

Use **single, lowercase words** for verbs. Path states the action; HTTP method enforces semantics.

**Standard CRUD verbs:**

| Verb | HTTP Method | Purpose | Example |
|---|---|---|---|
| `create` | `POST` | Create a new resource | `POST /orders/v1/create` |
| `read` | `GET` | Retrieve a resource by ID | `GET /orders/v1/read/{orderId}` |
| `update` | `PATCH` | Partially update a resource | `PATCH /orders/v1/update/{orderId}` |
| `list` | `POST` | List resources (simple) | `POST /orders/v1/list` |
| `search` | `POST` | Search with filters, sort, pagination | `POST /orders/v1/search` |
| `delete` | `DELETE` | Remove a resource | `DELETE /orders/v1/delete/{orderId}` |

> **Why `POST` for `list` and `search`?** Sunbird filters often include nested structures and arrays. `POST` keeps URL clean and avoids query-string length limits.
>
> **Why `PATCH` for `update`?** Sunbird convention is partial update. Use `PUT` only if user explicitly wants full-replacement semantics.

**Domain-specific verbs** — extend beyond CRUD when the business domain requires it:

| Verb | HTTP Method | Purpose | Example |
|---|---|---|---|
| `upload` | `POST` | Upload a file or binary | `POST /documents/v1/upload/{docId}` |
| `publish` | `POST` | Make a resource live | `POST /articles/v1/publish/{articleId}` |
| `review` | `POST` | Submit for review | `POST /articles/v1/review/{articleId}` |
| `reject` | `POST` | Reject under review | `POST /articles/v1/reject/{articleId}` |
| `retire` | `POST` | Soft-delete / retire | `POST /products/v1/retire/{productId}` |
| `copy` | `POST` | Duplicate a resource | `POST /templates/v1/copy/{templateId}` |
| `import` | `POST` | Import from external source | `POST /products/v1/import` |
| `flag` | `POST` | Flag for moderation | `POST /comments/v1/flag/{commentId}` |
| `flag/accept` | `POST` | Accept a flagged resource | `POST /comments/v1/flag/accept/{commentId}` |

### Naming Rules

1. **Use verbs in the URL.** Path states the action explicitly. Do not rely on HTTP method alone to convey intent.
2. **One verb = one responsibility.** Each endpoint does exactly one thing. `flag` only flags; `flag/accept` only accepts a flag.
3. **Lowercase, single-word verbs.** Use `create`, `publish`, `retire`. No hyphens or underscores in verbs.
4. **Compound actions use sub-verb nesting.** When a verb has variants, nest under the parent verb: `flag/accept`, `flag/reject`, `upload/url`. Sub-verb is part of the verb segment.
5. **Resource ID is always last.** `{resourceId}` is the final path segment, never elsewhere.
6. **Collection-level actions omit the ID.** `create`, `list`, `search`, `import` operate on the collection and take no ID.
7. **camelCase for path parameters and JSON fields.** Use `{orderId}`, not `{order_id}` or `{order-id}`. Never mix cases within one response.
8. **Timestamps are ISO 8601 UTC.** Every date/time field uses `2026-05-28T10:30:45Z` format.
9. **Singular vs plural — use both, consistently.** Intentional, not a contradiction. URL names the collection you act on; API ID and body name the entity type.

   | Place | Form | Example |
   |---|---|---|
   | URL resource segment | **plural** | `/orders/v1/create`, `/users/v1/read/{userId}` |
   | API ID (`id` field) | **singular** | `api.order.create`, `api.user.read` |
   | Request body inner key | **singular** | `"request": { "order": { ... } }` |
   | Response `result` — single item | **singular** | `"result": { "order": { ... } }` |
   | Response `result` — collection (`list`/`search`) | **plural** | `"result": { "count": 142, "orders": [...] }` |

   Matches the wider Sunbird ecosystem (`/content/v1/create` ↔ `api.content.create` ↔ `request.content`). Be consistent within one spec — do not flip a resource between `order` and `orders` in the same role.

---

## Request

### Headers

| Header | Required | Purpose |
|---|---|---|
| `Content-Type` | Yes (for body-bearing requests) | Media type of request body. Typically `application/json` |
| `Accept` | Yes | Expected response format. Typically `application/json` |
| `Authorization` | Yes | Bearer token or API key |

**Optional Sunbird headers** — include only when the endpoint needs them:

| Header | Purpose |
|---|---|
| `X-Channel-Id` | Identifies channel/tenant context for multi-tenant operations |
| `X-Authenticated-User-Token` | User-scoped token (in addition to platform `Authorization`) |
| `X-Source` | Identifies the calling client/application |
| `Idempotency-Key` | UUID supplied by client to safely retry `create` requests without duplicating resources |

If user has endpoint-specific custom headers, ask before generating the doc.

### Request Body

Every request body uses the full Sunbird envelope — mirrors the response envelope so the same shape works both directions. Wrap the payload inside `request`, with the **singular resource name** as inner key (see Naming Rules #9).

```json
{
  "id": "api.<resource>.<verb>",
  "ver": "<api_version>",
  "ts": "<ISO_8601_timestamp>",
  "params": {
    "msgid": "<client_generated_uuid>"
  },
  "request": {
    "<resource>": {
      "field1": "value1",
      "field2": "value2"
    }
  }
}
```

#### Envelope Field Reference

| Field | Type | Required | Purpose |
|---|---|---|---|
| `id` | string | Yes | Endpoint identifier in dot-notation. Must match URL (`api.order.create` ↔ `/orders/v1/create`). |
| `ver` | string | Yes | API version (e.g., `"1.0"`) — must match version in URL |
| `ts` | string | Yes | Client timestamp in ISO 8601 format (e.g., `2024-04-10T16:10:50+05:30` or `2026-05-28T10:30:45Z`) |
| `params.msgid` | string (UUID) | Yes | Client-generated UUID for this request. Server echoes it back in `response.params.msgid` for end-to-end traceability. |
| `request` | object | Yes | The payload. Contains the resource object for write operations, or filters/sort/fields for search. |

> **Why the full envelope?** Sunbird treats request and response symmetrically. Same logging, tracing, and version checks apply both directions. Clients generating `msgid` per request can correlate any failure end-to-end across logs.

**Example — Create an order:**

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:30:45+05:30",
  "params": {
    "msgid": "4a7f14c3-d61e-4d4f-be78-181834eeff6d"
  },
  "request": {
    "order": {
      "name": "Bulk Office Supplies",
      "type": "purchase",
      "priority": "high",
      "createdBy": "874ed8a5-782e-4f6c-8f36-e0288455901e"
    }
  }
}
```

**Example — Update an order:**

```json
{
  "id": "api.order.update",
  "ver": "1.0",
  "ts": "2026-05-28T10:35:18+05:30",
  "params": {
    "msgid": "5b8a25d4-e72f-5e5a-cf89-292945ffaa7e"
  },
  "request": {
    "order": {
      "versionKey": "1607631885207",
      "priority": "low",
      "description": "Updated priority"
    }
  }
}
```

> **`versionKey` rule:** Every update request must include the `versionKey` returned by the most recent read. Server rejects updates with a stale or missing `versionKey` (responds `409 CONFLICT`). Sunbird's optimistic concurrency mechanism.

### Search Request Body

For `search` endpoints, the envelope is identical — only the `request` payload changes. Use this standard shape inside `request`:

```json
{
  "id": "api.order.search",
  "ver": "1.0",
  "ts": "2026-05-28T10:32:01+05:30",
  "params": {
    "msgid": "6c9b36e5-f83a-6f6b-da9a-3a3a56aabb8f"
  },
  "request": {
    "filters": {
      "status": "active",
      "type": "purchase",
      "createdBy": "874ed8a5-782e-4f6c-8f36-e0288455901e"
    },
    "sort_by": { "createdOn": "desc" },
    "fields": ["identifier", "name", "status", "createdOn"],
    "limit": 25,
    "offset": 0
  }
}
```

`request` payload field reference:

| Field | Type | Purpose |
|---|---|---|
| `filters` | object | Equality filters; values can be scalars or arrays for IN-style match |
| `sort_by` | object | Map of field → `"asc"` or `"desc"` |
| `fields` | array | Whitelist of fields to return; omit for all fields |
| `limit` | int | Page size, default 25, max 100 |
| `offset` | int | Records to skip, default 0 |

> **Body-less requests (`GET` and `DELETE`):** `read` and `delete` use HTTP method + URL alone, no request body. `msgid` traceability supported via `X-Request-Id` header instead.

### Path Parameters

Path parameters represent specific resource instances and always appear as the **last segment** of the URL.

```
PATCH /orders/v1/update/{orderId}
```

- Use `camelCase`: `{orderId}`, `{userId}`, `{contentId}`.
- Never pass resource IDs in the request body — they belong in the URL.

### Query Parameters

Use query parameters **only** for read operations where the filter fits cleanly in a URL (e.g., a single status flag). For anything richer, use `search` with a request body.

---

## Response

### Response Envelope

Every response — success or failure — follows this structure:

```json
{
  "id": "api.<resource>.<verb>",
  "ver": "<api_version>",
  "ts": "<ISO_8601_timestamp>",
  "params": {
    "resmsgid": "<response_message_id>",
    "msgid": "<original_request_message_id>",
    "status": "successful | failed",
    "err": null,
    "errmsg": null
  },
  "responseCode": "<response_code>",
  "result": { }
}
```

### Field Reference

| Field | Type | Purpose |
|---|---|---|
| `id` | string | API endpoint identifier in dot-notation: `api.<resource>.<verb>`. Must match URL. |
| `ver` | string | API version (e.g., `"1.0"`) |
| `ts` | string | Server timestamp in ISO 8601 UTC |
| `params.resmsgid` | string (UUID) | Unique ID for **this response** — used for log correlation |
| `params.msgid` | string (UUID) \| null | Echoes request's `msgid` if client sent one, else `null` |
| `params.status` | string | `"successful"` or `"failed"` |
| `params.err` | string \| null | Machine-readable error code (e.g., `"ERR_FIELDS_MISSING"`). `null` on success. |
| `params.errmsg` | string \| null | Human-readable error description. `null` on success. |
| `responseCode` | string | High-level status — see HTTP mapping below |
| `result` | object | The response payload. Resource data on success, `{}` on failure. |

### Pagination in `result`

For `list` and `search` responses, `result` contains a count and a collection:

```json
"result": {
  "count": 142,
  "orders": [
    { "identifier": "...", "name": "..." },
    { "identifier": "...", "name": "..." }
  ]
}
```

The collection key is the **plural resource name** (`orders`, `users`, `assessments`).

### Success Response Example

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:30:45Z",
  "params": {
    "resmsgid": "3be02c4b-3324-41a3-afd8-60f6be0584d2",
    "msgid": "4a7f14c3-d61e-4d4f-be78-181834eeff6d",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "ord_2026052810304500001",
    "node_id": "ord_2026052810304500001",
    "versionKey": "1748428245207"
  }
}
```

---

## Error Responses

All errors use the same envelope. Only `responseCode`, `params.err`, `params.errmsg`, and HTTP status change. `result` always `{}` on failure.

### HTTP Status → `responseCode` Mapping

| HTTP | `responseCode` | When to use |
|---|---|---|
| `400` | `CLIENT_ERROR` | Invalid input, missing fields, validation failure |
| `401` | `UNAUTHORIZED` | Missing or invalid authentication |
| `403` | `FORBIDDEN` | Authenticated but not permitted |
| `404` | `RESOURCE_NOT_FOUND` | Resource does not exist |
| `405` | `METHOD_NOT_ALLOWED` | HTTP method not supported on this endpoint |
| `409` | `CONFLICT` | Version conflict, duplicate, or invalid state transition |
| `429` | `RATE_LIMIT_EXCEEDED` | Too many requests |
| `500` | `SERVER_ERROR` | Unexpected internal error |
| `502` | `BAD_GATEWAY` | Upstream returned invalid response |
| `503` | `SERVICE_UNAVAILABLE` | Server temporarily unavailable |
| `504` | `GATEWAY_TIMEOUT` | Upstream did not respond in time |

### Standard Error Codes

Use these `err` codes as defaults. Project-specific codes follow the `ERR_<DOMAIN>_<DETAIL>` shape.

| `err` code | HTTP | Meaning |
|---|---|---|
| `ERR_<RESOURCE>_FIELDS_MISSING` | 400 | Required fields missing in request |
| `ERR_<RESOURCE>_INVALID_VALUE` | 400 | Field value failed validation |
| `ERR_UNAUTHORIZED` | 401 | Token missing/invalid |
| `ERR_FORBIDDEN` | 403 | User lacks permission |
| `ERR_<RESOURCE>_NOT_FOUND` | 404 | Resource ID does not exist |
| `ERR_METHOD_NOT_ALLOWED` | 405 | Wrong HTTP method |
| `ERR_VERSION_CONFLICT` | 409 | Stale `versionKey` |
| `ERR_<RESOURCE>_DUPLICATE` | 409 | Resource already exists |
| `ERR_RATE_LIMIT_EXCEEDED` | 429 | Rate limit hit |
| `ERR_INTERNAL_SERVER_ERROR` | 500 | Generic server failure |
| `ERR_BAD_GATEWAY` | 502 | Upstream invalid response |
| `ERR_SERVICE_UNAVAILABLE` | 503 | Temporary outage |
| `ERR_GATEWAY_TIMEOUT` | 504 | Upstream timeout |

### Error Response Examples

For full JSON examples of every error response above, see [`references/error-catalog.md`](references/error-catalog.md). Read that file when documenting a specific endpoint's error responses.

> **Security note for 500 errors:** Never expose stack traces, SQL fragments, or internal field names in `errmsg`. Use a generic `"Something went wrong. Please try again."` and rely on `resmsgid` for server-side log correlation.

---

## Endpoint Documentation Template

Every endpoint in the final spec must follow this exact structure. Use as the template when documenting each endpoint from step 3 of the workflow.

```markdown
### <Action Name in Title Case>

**Description:** <One sentence stating what this endpoint does.>

| | |
|---|---|
| **URL** | `/orders/v1/create` |
| **Method** | `POST` |
| **API ID** | `api.order.create` |
| **Auth** | Required (Bearer token) |
| **Idempotent** | Yes / No |

#### Headers

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| Authorization | Bearer {token} | Yes |
| X-Channel-Id | {channelId} | Yes (multi-tenant) |

#### Path Parameters

| Name | Type | Required | Description |
|---|---|---|---|
| orderId | string | Yes | Unique identifier of the order |

(Omit this section if there are no path parameters.)

#### Request Body

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:30:45+05:30",
  "params": {
    "msgid": "4a7f14c3-d61e-4d4f-be78-181834eeff6d"
  },
  "request": {
    "order": {
      "name": "Bulk Office Supplies",
      "type": "purchase",
      "priority": "high"
    }
  }
}
```

#### Request Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| name | string | Yes | Display name of the order |
| type | string | Yes | One of: `purchase`, `transfer`, `return` |
| priority | string | No | One of: `low`, `medium`, `high`. Defaults to `medium`. |

#### Success Response — 200 OK

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:30:45Z",
  "params": {
    "resmsgid": "3be02c4b-3324-41a3-afd8-60f6be0584d2",
    "msgid": "4a7f14c3-d61e-4d4f-be78-181834eeff6d",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "ord_2026052810304500001",
    "versionKey": "1748428245207"
  }
}
```

#### Response Field Reference

| Field | Type | Description |
|---|---|---|
| result.identifier | string | The newly created order's unique ID |
| result.versionKey | string | Version token, required for any subsequent update |

#### Error Responses

| HTTP | `err` code | Cause |
|---|---|---|
| 400 | ERR_ORDER_FIELDS_MISSING | Required fields missing |
| 401 | ERR_UNAUTHORIZED | Token missing/invalid |
| 403 | ERR_FORBIDDEN | User lacks `order:create` permission |
| 409 | ERR_ORDER_DUPLICATE | Order with same idempotency key already exists |
| 500 | ERR_INTERNAL_SERVER_ERROR | Unexpected failure |

(Include one full JSON example for each error. See references/error-catalog.md.)

#### cURL Example

```bash
curl -X POST 'https://api.example.com/orders/v1/create' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <token>' \
  -d '{
    "id": "api.order.create",
    "ver": "1.0",
    "ts": "2026-05-28T10:30:45+05:30",
    "params": {
      "msgid": "4a7f14c3-d61e-4d4f-be78-181834eeff6d"
    },
    "request": {
      "order": {
        "name": "Bulk Office Supplies",
        "type": "purchase",
        "priority": "high"
      }
    }
  }'
```
```

---

## Red Flags

When designing or reviewing, watch for these — they signal the spec is drifting off-convention:

- Endpoints that return different shapes depending on conditions
- Inconsistent error formats across endpoints
- Field names mixing `camelCase` and `snake_case` in the same response
- Resource IDs in the request body instead of the URL path
- HTTP methods that don't match the verb (e.g., `GET /orders/v1/create`)
- Missing `versionKey` on update endpoints
- 500 responses that leak stack traces or internal field names
- `list` or `search` responses without a `count` field
- Timestamps in formats other than ISO 8601 UTC
- `id` field in the response not matching the URL (`api.order.create` vs `/orders/v1/create`)
- Singular resource name in URL (e.g., `/order/v1/create`) or plural in API ID (e.g., `api.orders.create`) — see Naming Rules #9
- Request body missing `id`, `ver`, `ts`, or `params.msgid` — full envelope is required, not just `request`

## Verification Checklist

Before delivering a spec, confirm each:

- [ ] Every endpoint has a consistent `resource/version/verb` URL
- [ ] Every endpoint's `id` field matches its URL (`api.order.create` ↔ `/orders/v1/create`)
- [ ] Request and response envelopes are consistent across all endpoints
- [ ] All applicable 4xx and 5xx errors are listed per endpoint
- [ ] Every endpoint has a cURL example
- [ ] All field names use a single case convention (camelCase recommended)
- [ ] Pagination is defined for any `list` or `search` endpoint (`count` + collection)
- [ ] Timestamps specify ISO 8601 UTC
- [ ] Update endpoints document the `versionKey` requirement
- [ ] Sample IDs in examples are realistic for the resource
- [ ] An `err` code is defined for every error row, following `ERR_<DOMAIN>_<DETAIL>`
- [ ] Auth requirements stated per endpoint, including role/permission notes
- [ ] Non-default headers (`X-Channel-Id`, `Idempotency-Key`, etc.) are documented where used
- [ ] Resource is **plural** in URLs and **singular** in API IDs, request bodies, and single-item response keys (Naming Rules #9)
- [ ] Every request body (for POST/PATCH endpoints) wraps the payload in the full envelope: `id`, `ver`, `ts`, `params.msgid`, `request`
- [ ] Request `id` matches response `id` matches URL (`api.order.create` everywhere)

## Reference Files

### Always read when applicable

- [`references/error-catalog.md`](references/error-catalog.md) — Full JSON examples for every standard error response. Read when filling in the **Error Responses** section of an endpoint.
- [`references/example-full-spec.md`](references/example-full-spec.md) — A complete worked example: an "Order" resource with all CRUD + search endpoints documented end-to-end. Read for a concrete reference of what a finished spec looks like.

### Optional extensions — read only when the user asks

The base skill above covers standard Sunbird API design (URL pattern, request/response envelope, CRUD + search, standard errors). Files below extend it for specific patterns. **Do not surface these proactively** — read only when user explicitly requests the corresponding capability.

- [`references/state-workflow.md`](references/state-workflow.md) — Draft → Review → Live → Retired state machine, the `review`/`publish`/`reject`/`retire` transition endpoints, and the `ERR_INVALID_STATE_TRANSITION` error. Read **only when user explicitly says** their resource needs a review/approval lifecycle, mentions Draft/Review/Live/Retired states, or asks about `publish`/`review`/`reject`/`retire` endpoints. For plain CRUD designs, ignore this file.