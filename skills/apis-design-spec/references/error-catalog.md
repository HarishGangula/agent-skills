# Error Response Catalog

Complete JSON examples for every standard error response. Use these as templates when documenting an endpoint's error responses — copy the relevant block and adjust `id`, `err`, and `errmsg` to fit the specific endpoint and condition.

All examples assume the endpoint is `api.order.<verb>`. Replace `order` with the actual resource name.

---

## 400 — Bad Request

Missing required fields, invalid values, or malformed request body.

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "c169a7a0-3ac4-11eb-b0a2-8d5c9f561887",
    "msgid": null,
    "status": "failed",
    "err": "ERR_ORDER_FIELDS_MISSING",
    "errmsg": "Required fields for create order are missing: name, type"
  },
  "responseCode": "CLIENT_ERROR",
  "result": {}
}
```

## 401 — Unauthorized

No token provided, or the token is expired/invalid.

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "a21b3c40-4e12-11eb-a1b2-3c4d5e6f7890",
    "msgid": null,
    "status": "failed",
    "err": "ERR_UNAUTHORIZED",
    "errmsg": "Access denied. Authentication token is missing or invalid"
  },
  "responseCode": "UNAUTHORIZED",
  "result": {}
}
```

## 403 — Forbidden

The user is authenticated but does not have permission for this action.

```json
{
  "id": "api.order.update",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "b32c4d50-5f23-11eb-c2d3-4e5f6a7b8c90",
    "msgid": null,
    "status": "failed",
    "err": "ERR_FORBIDDEN",
    "errmsg": "You do not have permission to update this order"
  },
  "responseCode": "FORBIDDEN",
  "result": {}
}
```

## 404 — Resource Not Found

The resource ID in the URL does not match any existing resource.

```json
{
  "id": "api.order.read",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "5b4f0b33-3941-4c18-b8bf-123c2e0348e6",
    "msgid": null,
    "status": "failed",
    "err": "ERR_ORDER_NOT_FOUND",
    "errmsg": "Order does not exist. Invalid order ID: ord_999999"
  },
  "responseCode": "RESOURCE_NOT_FOUND",
  "result": {}
}
```

## 405 — Method Not Allowed

The HTTP method used is not supported on this endpoint.

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "d54e6f70-7a34-11eb-e4f5-6a7b8c9d0e12",
    "msgid": null,
    "status": "failed",
    "err": "ERR_METHOD_NOT_ALLOWED",
    "errmsg": "GET method is not supported for this endpoint. Use POST"
  },
  "responseCode": "METHOD_NOT_ALLOWED",
  "result": {}
}
```

## 409 — Conflict (version conflict)

The request conflicts with the current state of the resource — typically a stale `versionKey`.

```json
{
  "id": "api.order.update",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "e65f7a80-8b45-11eb-f5a6-7b8c9d0e1f23",
    "msgid": null,
    "status": "failed",
    "err": "ERR_VERSION_CONFLICT",
    "errmsg": "Resource has been modified. Fetch the latest version and retry"
  },
  "responseCode": "CONFLICT",
  "result": {}
}
```

## 409 — Conflict (duplicate)

A resource with the same idempotency key or unique constraint already exists.

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "e76f8a90-9c56-11eb-a6b7-8c9d0e1f2a34",
    "msgid": null,
    "status": "failed",
    "err": "ERR_ORDER_DUPLICATE",
    "errmsg": "An order with the same idempotency key already exists"
  },
  "responseCode": "CONFLICT",
  "result": {}
}
```

## 429 — Too Many Requests

The client has sent too many requests in a given time window.

```json
{
  "id": "api.order.list",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "f76a8b90-9c56-11eb-a6b7-8c9d0e1f2a34",
    "msgid": null,
    "status": "failed",
    "err": "ERR_RATE_LIMIT_EXCEEDED",
    "errmsg": "Rate limit exceeded. Retry after 30 seconds"
  },
  "responseCode": "RATE_LIMIT_EXCEEDED",
  "result": {}
}
```

## 500 — Internal Server Error

An unexpected error occurred on the server. Never expose stack traces or internal details — use a generic message and rely on `resmsgid` for log correlation.

```json
{
  "id": "api.order.create",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "f234a6f0-3ac4-11eb-b0a2-8d5c9f561887",
    "msgid": null,
    "status": "failed",
    "err": "ERR_INTERNAL_SERVER_ERROR",
    "errmsg": "Something went wrong. Please try again."
  },
  "responseCode": "SERVER_ERROR",
  "result": {}
}
```

## 502 — Bad Gateway

An upstream dependency returned an invalid response.

```json
{
  "id": "api.order.publish",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "a87b9ca0-ad67-11eb-b7c8-9d0e1f2a3b45",
    "msgid": null,
    "status": "failed",
    "err": "ERR_BAD_GATEWAY",
    "errmsg": "Upstream service returned an invalid response"
  },
  "responseCode": "BAD_GATEWAY",
  "result": {}
}
```

## 503 — Service Unavailable

The server is temporarily unable to handle the request.

```json
{
  "id": "api.order.read",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "b98cada0-be78-11eb-c8d9-0e1f2a3b4c56",
    "msgid": null,
    "status": "failed",
    "err": "ERR_SERVICE_UNAVAILABLE",
    "errmsg": "Service is temporarily unavailable. Please retry after some time"
  },
  "responseCode": "SERVICE_UNAVAILABLE",
  "result": {}
}
```

## 504 — Gateway Timeout

An upstream dependency did not respond within the expected time.

```json
{
  "id": "api.order.publish",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:12Z",
  "params": {
    "resmsgid": "ca9debe0-cf89-11eb-d9ea-1f2a3b4c5d67",
    "msgid": null,
    "status": "failed",
    "err": "ERR_GATEWAY_TIMEOUT",
    "errmsg": "Upstream service did not respond in time"
  },
  "responseCode": "GATEWAY_TIMEOUT",
  "result": {}
}
```