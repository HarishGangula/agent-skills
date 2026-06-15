# Example: Complete Order Resource API Spec

A fully-worked reference showing what a finished Sunbird API spec looks like for a single resource with full CRUD + search. Use as the target shape when documenting a new resource — match section ordering, table style, and depth of detail.

---

# Order Service API Specification

**Version:** 1.0
**Base URL:** `https://api.example.com`

## Overview

The Order Service manages purchase, transfer, and return orders for the platform. All endpoints follow the Sunbird envelope convention.

## Authentication

All endpoints require a Bearer token in the `Authorization` header. Multi-tenant deployments also require `X-Channel-Id`.

## Endpoints

| # | Method | URL | API ID | Purpose |
|---|---|---|---|---|
| 1 | POST | `/orders/v1/create` | `api.order.create` | Create a new order |
| 2 | GET | `/orders/v1/read/{orderId}` | `api.order.read` | Fetch an order by ID |
| 3 | POST | `/orders/v1/search` | `api.order.search` | Search orders with filters |
| 4 | PATCH | `/orders/v1/update/{orderId}` | `api.order.update` | Partially update an order |
| 5 | DELETE | `/orders/v1/delete/{orderId}` | `api.order.delete` | Delete an order |

---

### 1. Create Order

**Description:** Creates a new order in the system.

| | |
|---|---|
| **URL** | `/orders/v1/create` |
| **Method** | `POST` |
| **API ID** | `api.order.create` |
| **Auth** | Required (Bearer token) |
| **Idempotent** | Yes (when `Idempotency-Key` is supplied) |

#### Headers

| Header | Value | Required |
|---|---|---|
| Content-Type | application/json | Yes |
| Authorization | Bearer {token} | Yes |
| X-Channel-Id | {channelId} | Yes |
| Idempotency-Key | {uuid} | No |

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
      "priority": "high",
      "createdBy": "874ed8a5-782e-4f6c-8f36-e0288455901e"
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
| createdBy | string (UUID) | Yes | User ID of the creator |

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
| 409 | ERR_ORDER_DUPLICATE | Same idempotency key already used |
| 500 | ERR_INTERNAL_SERVER_ERROR | Unexpected failure |

#### cURL Example

```bash
curl -X POST 'https://api.example.com/orders/v1/create' \
  -H 'Content-Type: application/json' \
  -H 'Authorization: Bearer <token>' \
  -H 'X-Channel-Id: <channelId>' \
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
        "priority": "high",
        "createdBy": "874ed8a5-782e-4f6c-8f36-e0288455901e"
      }
    }
  }'
```

---

### 2. Read Order

**Description:** Fetches a single order by its identifier.

| | |
|---|---|
| **URL** | `/orders/v1/read/{orderId}` |
| **Method** | `GET` |
| **API ID** | `api.order.read` |
| **Auth** | Required (Bearer token) |
| **Idempotent** | Yes |

#### Path Parameters

| Name | Type | Required | Description |
|---|---|---|---|
| orderId | string | Yes | Unique identifier of the order |

#### Headers

| Header | Value | Required |
|---|---|---|
| Authorization | Bearer {token} | Yes |
| X-Channel-Id | {channelId} | Yes |

#### Success Response — 200 OK

```json
{
  "id": "api.order.read",
  "ver": "1.0",
  "ts": "2026-05-28T10:31:10Z",
  "params": {
    "resmsgid": "4cf13d5c-4435-42b4-9c47-71f7cf059eda",
    "msgid": "9a2c69d8-2b6d-4c9e-fd2d-6d6d89dde9b2",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "order": {
      "identifier": "ord_2026052810304500001",
      "name": "Bulk Office Supplies",
      "type": "purchase",
      "priority": "high",
      "status": "active",
      "createdBy": "874ed8a5-782e-4f6c-8f36-e0288455901e",
      "createdOn": "2026-05-28T10:30:45Z",
      "lastUpdatedOn": "2026-05-28T10:30:45Z",
      "versionKey": "1748428245207"
    }
  }
}
```

#### Error Responses

| HTTP | `err` code | Cause |
|---|---|---|
| 401 | ERR_UNAUTHORIZED | Token missing/invalid |
| 403 | ERR_FORBIDDEN | User lacks `order:read` permission |
| 404 | ERR_ORDER_NOT_FOUND | Order does not exist |
| 500 | ERR_INTERNAL_SERVER_ERROR | Unexpected failure |

#### cURL Example

```bash
curl -X GET 'https://api.example.com/orders/v1/read/ord_2026052810304500001' \
  -H 'Authorization: Bearer <token>' \
  -H 'X-Channel-Id: <channelId>' \
  -H 'X-Request-Id: 9a2c69d8-2b6d-4c9e-fd2d-6d6d89dde9b2'
```

> For body-less requests (GET, DELETE), use the `X-Request-Id` header to supply a UUID for traceability. Server echoes it in `response.params.msgid`.

---

### 3. Search Orders

**Description:** Search orders with filters, sort, and pagination.

| | |
|---|---|
| **URL** | `/orders/v1/search` |
| **Method** | `POST` |
| **API ID** | `api.order.search` |
| **Auth** | Required (Bearer token) |
| **Idempotent** | Yes |

#### Request Body

```json
{
  "id": "api.order.search",
  "ver": "1.0",
  "ts": "2026-05-28T10:32:01+05:30",
  "params": {
    "msgid": "6c9b36e5-f83a-4f6b-da9a-3a3a56aabb8f"
  },
  "request": {
    "filters": {
      "status": "active",
      "type": ["purchase", "transfer"]
    },
    "sort_by": { "createdOn": "desc" },
    "fields": ["identifier", "name", "status", "createdOn"],
    "limit": 25,
    "offset": 0
  }
}
```

#### Request Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| filters | object | No | Equality filters. Values can be scalars or arrays. |
| sort_by | object | No | Map of field → `"asc"` or `"desc"` |
| fields | array | No | Whitelist of fields to return |
| limit | int | No | Page size (default 25, max 100) |
| offset | int | No | Records to skip (default 0) |

#### Success Response — 200 OK

```json
{
  "id": "api.order.search",
  "ver": "1.0",
  "ts": "2026-05-28T10:32:01Z",
  "params": {
    "resmsgid": "5d124e6d-5546-43c5-ad58-82c8d6160feb",
    "msgid": "6c9b36e5-f83a-4f6b-da9a-3a3a56aabb8f",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "count": 142,
    "orders": [
      {
        "identifier": "ord_2026052810304500001",
        "name": "Bulk Office Supplies",
        "status": "active",
        "createdOn": "2026-05-28T10:30:45Z"
      },
      {
        "identifier": "ord_2026052810304500002",
        "name": "Quarterly Stationery",
        "status": "active",
        "createdOn": "2026-05-28T09:15:22Z"
      }
    ]
  }
}
```

#### Error Responses

| HTTP | `err` code | Cause |
|---|---|---|
| 400 | ERR_ORDER_INVALID_VALUE | Invalid filter or sort field |
| 401 | ERR_UNAUTHORIZED | Token missing/invalid |
| 500 | ERR_INTERNAL_SERVER_ERROR | Unexpected failure |

---

### 4. Update Order

**Description:** Partially updates an existing order. Requires the current `versionKey`.

| | |
|---|---|
| **URL** | `/orders/v1/update/{orderId}` |
| **Method** | `PATCH` |
| **API ID** | `api.order.update` |
| **Auth** | Required (Bearer token) |
| **Idempotent** | Yes (versionKey ensures safe retry) |

#### Path Parameters

| Name | Type | Required | Description |
|---|---|---|---|
| orderId | string | Yes | Unique identifier of the order |

#### Request Body

```json
{
  "id": "api.order.update",
  "ver": "1.0",
  "ts": "2026-05-28T10:35:18+05:30",
  "params": {
    "msgid": "7e0a47b6-094b-4a7c-eb0b-4b4b67bbcc90"
  },
  "request": {
    "order": {
      "versionKey": "1748428245207",
      "priority": "low",
      "description": "Reduced priority after review"
    }
  }
}
```

#### Request Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| versionKey | string | Yes | Most recently read versionKey for optimistic concurrency |
| priority | string | No | One of: `low`, `medium`, `high` |
| description | string | No | Free-text description |

#### Success Response — 200 OK

```json
{
  "id": "api.order.update",
  "ver": "1.0",
  "ts": "2026-05-28T10:35:18Z",
  "params": {
    "resmsgid": "6e235f7e-6657-44d6-be69-93b9ea271fcc",
    "msgid": "7e0a47b6-094b-4a7c-eb0b-4b4b67bbcc90",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "ord_2026052810304500001",
    "versionKey": "1748428518104"
  }
}
```

#### Error Responses

| HTTP | `err` code | Cause |
|---|---|---|
| 400 | ERR_ORDER_FIELDS_MISSING | versionKey missing |
| 401 | ERR_UNAUTHORIZED | Token missing/invalid |
| 403 | ERR_FORBIDDEN | User lacks `order:update` permission |
| 404 | ERR_ORDER_NOT_FOUND | Order does not exist |
| 409 | ERR_VERSION_CONFLICT | Stale versionKey — fetch latest and retry |
| 500 | ERR_INTERNAL_SERVER_ERROR | Unexpected failure |

---

### 5. Delete Order

**Description:** Permanently removes an order.

| | |
|---|---|
| **URL** | `/orders/v1/delete/{orderId}` |
| **Method** | `DELETE` |
| **API ID** | `api.order.delete` |
| **Auth** | Required (Bearer token) |
| **Idempotent** | Yes |

#### Path Parameters

| Name | Type | Required | Description |
|---|---|---|---|
| orderId | string | Yes | Unique identifier of the order |

#### Success Response — 200 OK

```json
{
  "id": "api.order.delete",
  "ver": "1.0",
  "ts": "2026-05-28T10:40:00Z",
  "params": {
    "resmsgid": "7f146a8f-7768-45e7-cf7a-04a0fa382ddd",
    "msgid": "8f1b58c7-1a5c-4b8d-fc1c-5c5c78ccdda1",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "ord_2026052810304500001"
  }
}
```

#### Error Responses

| HTTP | `err` code | Cause |
|---|---|---|
| 401 | ERR_UNAUTHORIZED | Token missing/invalid |
| 403 | ERR_FORBIDDEN | User lacks `order:delete` permission |
| 404 | ERR_ORDER_NOT_FOUND | Order does not exist |
| 500 | ERR_INTERNAL_SERVER_ERROR | Unexpected failure |

---

## Error Code Reference

Consolidated list of every `err` code used in this spec:

| `err` code | HTTP | Used in |
|---|---|---|
| ERR_ORDER_FIELDS_MISSING | 400 | create, update |
| ERR_ORDER_INVALID_VALUE | 400 | search |
| ERR_UNAUTHORIZED | 401 | all |
| ERR_FORBIDDEN | 403 | all |
| ERR_ORDER_NOT_FOUND | 404 | read, update, delete |
| ERR_VERSION_CONFLICT | 409 | update |
| ERR_ORDER_DUPLICATE | 409 | create |
| ERR_INTERNAL_SERVER_ERROR | 500 | all |

## Changelog

| Version | Date | Changes |
|---|---|---|
| 1.0 | 2026-05-28 | Initial release |