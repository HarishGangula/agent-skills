---
name: apis-design-spec
description: This Guides the designing of the APIs, Use this when designing the restful APIs in standard format. Defining the contracts between backend and frontend systems and between microservices to communicate clearly.
license: MIT
metadata:
    author: Harish Kumar Gangula
    version: "1.0"
---

# API Design Specification

## Overview

Design the Restful APIs which follows the standard and easy to understand , implement and maintain. Provide clear contracts and communicate clearly. It should be clear and extensible to support needs for the future enhancements.

## When to Use

-  Designing new API endpoints
-  Revising or improving existing APIs
-  Developing API documentation
-  Reviewing API implementations for compliance with best practices

## API Patterns

### Naming

Each API endpoint is responsible for **one capability**. The URL path should clearly communicate **what action** is being performed on **which resource**.

#### URL Pattern

```
/{resource}/{version}/{verb}/{resourceId}
```

| Segment        | Purpose                                    | Example             |
|----------------|--------------------------------------------|---------------------|
| **Resource**   | The domain entity the API manages          | `orders`, `users`   |
| **Version**    | The API version                            | `v1`, `v2`          |
| **Verb**       | The action being performed on the resource | `create`, `read`    |
| **Identifier** | (Optional) The specific resource instance  | `{orderId}`         |

#### Verb to HTTP Method Mapping

Each verb pairs with the appropriate HTTP method. Use **single, lowercase words** for verbs.

**Standard CRUD verbs:**

| Verb      | HTTP Method | Purpose                           | Example                             |
|-----------|-------------|-----------------------------------|-------------------------------------|
| `create`  | `POST`      | Create a new resource             | `POST /orders/v1/create`            |
| `read`    | `GET`       | Retrieve a resource by ID         | `GET /orders/v1/read/{orderId}`     |
| `update`  | `PATCH`     | Partially update a resource       | `PATCH /orders/v1/update/{orderId}` |
| `list`    | `POST`      | List resources with filters       | `POST /orders/v1/list`              |
| `delete`  | `DELETE`    | Remove a resource                 | `DELETE /orders/v1/delete/{orderId}`|

**Domain-specific verbs** — extend beyond CRUD when the business domain requires it:

| Verb            | HTTP Method | Purpose                                | Example                                    |
|-----------------|-------------|----------------------------------------|--------------------------------------------|
| `upload`        | `POST`      | Upload a file or binary for a resource | `POST /documents/v1/upload/{docId}`        |
| `publish`       | `POST`      | Make a resource live/available         | `POST /articles/v1/publish/{articleId}`    |
| `review`        | `POST`      | Submit a resource for review           | `POST /articles/v1/review/{articleId}`     |
| `reject`        | `POST`      | Reject a resource under review         | `POST /articles/v1/reject/{articleId}`     |
| `retire`        | `POST`      | Soft-delete / retire a resource        | `POST /products/v1/retire/{productId}`     |
| `copy`          | `POST`      | Duplicate a resource                   | `POST /templates/v1/copy/{templateId}`     |
| `import`        | `POST`      | Import from an external source         | `POST /products/v1/import`                 |
| `flag`          | `POST`      | Flag for moderation review             | `POST /comments/v1/flag/{commentId}`       |
| `flag/accept`   | `POST`      | Accept a flagged resource              | `POST /comments/v1/flag/accept/{commentId}`|

#### Naming Rules

1. **Use verbs in the URL** — The path should state the action explicitly (`create`, `publish`, `retire`), not rely on the HTTP method alone.
2. **One verb = One responsibility** — Each endpoint does exactly one thing. `flag` only flags; `flag/accept` only accepts a flag.
3. **Lowercase, single-word verbs** — Use `create`, `read`, `publish`, `discard`. Avoid hyphens or underscores in verbs.
4. **Compound actions use path nesting** — When a verb has sub-actions, nest them: `flag/accept`, `flag/reject`, `upload/url`.
5. **Resource ID comes last** — `{resourceId}` is always the final path segment.
6. **Collection-level actions omit the ID** — Actions like `create`, `list`, and `import` that operate on the collection don't need an ID.


### Request

Every request should follow a consistent structure across all endpoints.

#### Headers

| Header                       | Required | Purpose                                                        |
|------------------------------|----------|----------------------------------------------------------------|
| `Content-Type`               | Yes      | Media type of the request body. Typically `application/json`   |
| `Accept`                     | Yes      | Expected response format. Typically `application/json`         |
| `Authorization`              | Yes      | Auth token or API key for the request                          |
| `X-Authenticated-User-Token` | Yes      | Token identifying the authenticated user performing the action |
| `X-Channel-Id`               | No       | Identifies the tenant/channel the user belongs to              |

#### Request Body

Wrap the request payload in a standard envelope: `request` → `<resource>`.

```json
{
  "request": {
    "<resource>": {
      "field1": "value1",
      "field2": "value2"
    }
  }
}
```

**Example — Create an order:**

```json
{
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
  "request": {
    "order": {
      "versionKey": "1607631885207",
      "priority": "low",
      "description": "Updated priority"
    }
  }
}
```

> **Note:** The `versionKey` must be provided on update requests to ensure optimistic concurrency — the server rejects the update if the version has changed since the last read.

#### Path Parameters

Path parameters represent specific resource instances. They always appear as the **last segment** of the URL.

```
PATCH /orders/v1/update/{orderId}
```

- Use `camelCase` for parameter names: `{orderId}`, `{userId}`, `{contentId}`
- Never pass resource IDs in the request body — they belong in the URL path

#### Query Parameters

Use query parameters only for **read/list** operations to control filtering, pagination, and field selection.

```
POST /orders/v1/list?limit=10&offset=0
```

---

### Response

Every response follows a **standard envelope** structure, regardless of success or failure.

#### Response Envelope

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

#### Field Reference

| Field              | Type     | Purpose                                                                 |
|--------------------|----------|-------------------------------------------------------------------------|
| `id`               | `string` | Identifies the API endpoint in dot-notation: `api.<resource>.<verb>`    |
| `ver`              | `string` | API version that produced this response (e.g. `"1.0"`, `"3.0"`)        |
| `ts`               | `string` | Server timestamp in ISO 8601 format when the response was generated     |
| `params.resmsgid`  | `string` | Unique ID generated for **this response** (for logging and tracing)     |
| `params.msgid`     | `string` | Original request message ID echoed back (for end-to-end traceability)   |
| `params.status`    | `string` | `"successful"` or `"failed"`                                           |
| `params.err`       | `string` | Machine-readable error code (e.g. `"ERR_FIELDS_MISSING"`). `null` on success |
| `params.errmsg`    | `string` | Human-readable error description. `null` on success                     |
| `responseCode`     | `string` | High-level status: `"OK"`, `"CLIENT_ERROR"`, `"RESOURCE_NOT_FOUND"`, `"SERVER_ERROR"` |
| `result`           | `object` | The actual response payload. Contains the resource data on success, empty `{}` on failure |

#### Success Response Example

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2025-03-15T10:30:45Z",
  "params": {
    "resmsgid": "3be02c4b-3324-41a3-afd8-60f6be0584d2",
    "msgid": null,
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "ord_2025031510304500001",
    "node_id": "ord_2025031510304500001",
    "versionKey": "1607631885207"
  }
}
```

#### Error Response Example

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2025-03-15T10:31:12Z",
  "params": {
    "resmsgid": "c169a7a0-3ac4-11eb-b0a2-8d5c9f561887",
    "msgid": null,
    "status": "failed",
    "err": "ERR_ORDER_FIELDS_MISSING",
    "errmsg": "Required fields for create order are missing"
  },
  "responseCode": "CLIENT_ERROR",
  "result": {}
}
```

### Errors

### Examples




## Red  Flags


## Verification


Output format examples

Check List for each HTTP method type



URL patterns

Verb , action

Content,  read, create, update, delete, list

Request body

Response body

Different error status 
4XX, 5XX errors

Headers 
Accept, Content-Type, Authorization
