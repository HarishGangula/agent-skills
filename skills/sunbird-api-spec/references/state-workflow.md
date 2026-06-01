# State Workflow — Draft → Review → Live

Many Sunbird resources (content, datasets, frameworks, articles) follow a multi-state lifecycle rather than being immediately public on creation. This file documents the standard state machine and the endpoints that drive transitions between states.

Use this reference whenever a resource needs more than just CRUD — when it must be reviewed, approved, published, or retired through a defined workflow.

---

## State Machine

```
                  ┌────────────────────────────────┐
                  ▼                                │
              ┌───────┐  review   ┌────────┐      │ reject
   create ──▶ │ Draft │──────────▶│ Review │──────┘
              └───────┘           └────────┘
                                      │
                                      │ publish
                                      ▼
                                  ┌──────┐  retire   ┌─────────┐
                                  │ Live │──────────▶│ Retired │
                                  └──────┘           └─────────┘
```

## States

| State | Meaning | Visible to public? |
|---|---|---|
| **Draft** | Resource has been created and is being edited. Owner can update freely. | No |
| **Review** | Resource has been submitted for review. Editing is locked; awaiting moderator decision. | No |
| **Live** | Resource has been published. Publicly listed and searchable. | Yes |
| **Retired** | Resource has been removed from public listings. Archived for record-keeping; not searchable. | No |

## Transitions

| Verb | URL | HTTP | From state | To state | Performed by |
|---|---|---|---|---|---|
| `create` | `POST /<resource>/v1/create` | POST | — | Draft | Owner |
| `update` | `PATCH /<resource>/v1/update/{id}` | PATCH | Draft (only) | Draft | Owner |
| `review` | `POST /<resource>/v1/review/{id}` | POST | Draft | Review | Owner |
| `publish` | `POST /<resource>/v1/publish/{id}` | POST | Review | Live | Reviewer / moderator |
| `reject` | `POST /<resource>/v1/reject/{id}` | POST | Review | Draft | Reviewer / moderator |
| `retire` | `POST /<resource>/v1/retire/{id}` | POST | Live | Retired | Owner or moderator |

> **Rule:** Any attempt to invoke a transition from a state that is not its `From state` returns `409 CONFLICT` with `err: "ERR_INVALID_STATE_TRANSITION"`.

---

## Endpoint Examples

Each transition endpoint follows the standard Sunbird envelope. The resource ID is in the URL; the body carries any transition-specific data (reviewer comments, rejection reason, retirement note).

### Review — submit Draft for review

**URL:** `POST /articles/v1/review/{articleId}`
**API ID:** `api.article.review`
**Performed by:** Owner

#### Request Body

```json
{
  "id": "api.article.review",
  "ver": "1.0",
  "ts": "2026-05-28T11:00:00+05:30",
  "params": {
    "msgid": "5b8a25d4-e72f-4e5a-cf89-292945ffaa7e"
  },
  "request": {
    "article": {
      "versionKey": "1748428245207"
    }
  }
}
```

#### Success Response — 200 OK

```json
{
  "id": "api.article.review",
  "ver": "1.0",
  "ts": "2026-05-28T11:00:00Z",
  "params": {
    "resmsgid": "3be02c4b-3324-41a3-afd8-60f6be0584d2",
    "msgid": "5b8a25d4-e72f-4e5a-cf89-292945ffaa7e",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "art_2026052810304500001",
    "status": "Review",
    "versionKey": "1748428800120"
  }
}
```

### Publish — promote Review to Live

**URL:** `POST /articles/v1/publish/{articleId}`
**API ID:** `api.article.publish`
**Performed by:** Reviewer

#### Request Body

```json
{
  "id": "api.article.publish",
  "ver": "1.0",
  "ts": "2026-05-28T11:15:00+05:30",
  "params": {
    "msgid": "6c9b36e5-f83a-4f6b-da9a-3a3a56aabb8f"
  },
  "request": {
    "article": {
      "versionKey": "1748428800120",
      "publishedBy": "rvw_874ed8a5-782e-4f6c-8f36-e0288455901e",
      "publishComment": "Reviewed and approved for publication"
    }
  }
}
```

#### Success Response — 200 OK

```json
{
  "id": "api.article.publish",
  "ver": "1.0",
  "ts": "2026-05-28T11:15:00Z",
  "params": {
    "resmsgid": "4cf13d5c-4435-42b4-9c47-71f7cf059eda",
    "msgid": "6c9b36e5-f83a-4f6b-da9a-3a3a56aabb8f",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "art_2026052810304500001",
    "status": "Live",
    "publishedOn": "2026-05-28T11:15:00Z",
    "versionKey": "1748429700240"
  }
}
```

### Reject — return Review to Draft

**URL:** `POST /articles/v1/reject/{articleId}`
**API ID:** `api.article.reject`
**Performed by:** Reviewer

A rejection must include a `rejectComment` so the owner knows what to fix.

#### Request Body

```json
{
  "id": "api.article.reject",
  "ver": "1.0",
  "ts": "2026-05-28T11:20:00+05:30",
  "params": {
    "msgid": "7e0a47b6-094b-4a7c-eb0b-4b4b67bbcc90"
  },
  "request": {
    "article": {
      "versionKey": "1748428800120",
      "rejectedBy": "rvw_874ed8a5-782e-4f6c-8f36-e0288455901e",
      "rejectComment": "Title needs to be clearer; section 3 has factual issues",
      "rejectReasons": ["incomplete_content", "factual_inaccuracy"]
    }
  }
}
```

#### Request Field Reference

| Field | Type | Required | Description |
|---|---|---|---|
| versionKey | string | Yes | Most recently read versionKey |
| rejectedBy | string | Yes | Reviewer user ID |
| rejectComment | string | Yes | Free-text feedback for the owner |
| rejectReasons | array | No | Optional structured reason codes for analytics |

#### Success Response — 200 OK

```json
{
  "id": "api.article.reject",
  "ver": "1.0",
  "ts": "2026-05-28T11:20:00Z",
  "params": {
    "resmsgid": "5d124e6d-5546-43c5-ad58-82c8d6160feb",
    "msgid": "7e0a47b6-094b-4a7c-eb0b-4b4b67bbcc90",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "art_2026052810304500001",
    "status": "Draft",
    "versionKey": "1748430000310"
  }
}
```

### Retire — remove Live from public listings

**URL:** `POST /articles/v1/retire/{articleId}`
**API ID:** `api.article.retire`
**Performed by:** Owner or moderator

#### Request Body

```json
{
  "id": "api.article.retire",
  "ver": "1.0",
  "ts": "2026-05-28T15:00:00+05:30",
  "params": {
    "msgid": "8f1b58c7-1a5c-4b8d-fc1c-5c5c78ccdda1"
  },
  "request": {
    "article": {
      "versionKey": "1748429700240",
      "retiredBy": "874ed8a5-782e-4f6c-8f36-e0288455901e",
      "retireReason": "Superseded by article art_2026052899999900001"
    }
  }
}
```

#### Success Response — 200 OK

```json
{
  "id": "api.article.retire",
  "ver": "1.0",
  "ts": "2026-05-28T15:00:00Z",
  "params": {
    "resmsgid": "6e235f7e-6657-44d6-be69-93b9ea271fcc",
    "msgid": "8f1b58c7-1a5c-4b8d-fc1c-5c5c78ccdda1",
    "err": null,
    "status": "successful",
    "errmsg": null
  },
  "responseCode": "OK",
  "result": {
    "identifier": "art_2026052810304500001",
    "status": "Retired",
    "retiredOn": "2026-05-28T15:00:00Z",
    "versionKey": "1748443200400"
  }
}
```

---

## Error Responses for Workflow Endpoints

In addition to the standard error codes (`ERR_UNAUTHORIZED`, `ERR_FORBIDDEN`, `ERR_<RESOURCE>_NOT_FOUND`, `ERR_VERSION_CONFLICT`, `ERR_INTERNAL_SERVER_ERROR`), workflow endpoints have one workflow-specific code:

### 409 — Invalid State Transition

Triggered when a transition is invoked from a state that does not allow it (e.g., trying to publish a Draft directly, retiring an already-retired resource).

```json
{
  "id": "api.article.publish",
  "ver": "1.0",
  "ts": "2026-05-28T11:15:00Z",
  "params": {
    "resmsgid": "9a2c69d8-2b6d-4c9e-fd2d-6d6d89dde9b2",
    "msgid": "6c9b36e5-f83a-4f6b-da9a-3a3a56aabb8f",
    "status": "failed",
    "err": "ERR_INVALID_STATE_TRANSITION",
    "errmsg": "Cannot publish article: current state is 'Draft', expected 'Review'"
  },
  "responseCode": "CONFLICT",
  "result": {}
}
```

The `errmsg` should always state both the current state and the expected state so the client can present a clear message to the user.

### Common Per-Transition Errors

| Transition | Most likely error | Cause |
|---|---|---|
| review | ERR_INVALID_STATE_TRANSITION | Resource is not in Draft |
| publish | ERR_INVALID_STATE_TRANSITION | Resource is not in Review |
| publish | ERR_FORBIDDEN | Caller does not have reviewer role |
| reject | ERR_<RESOURCE>_FIELDS_MISSING | `rejectComment` is missing |
| reject | ERR_INVALID_STATE_TRANSITION | Resource is not in Review |
| retire | ERR_INVALID_STATE_TRANSITION | Resource is not Live (already retired, or still Draft/Review) |

---

## Documenting a Workflow-Enabled Resource

When the resource being designed follows this lifecycle, the enumerate-endpoints table in step 2 of the SKILL.md workflow should include the transition endpoints alongside the CRUD endpoints. Example:

| # | Method | URL | API ID | Purpose |
|---|---|---|---|---|
| 1 | POST | `/articles/v1/create` | `api.article.create` | Create a new article (Draft) |
| 2 | GET | `/articles/v1/read/{articleId}` | `api.article.read` | Fetch an article |
| 3 | PATCH | `/articles/v1/update/{articleId}` | `api.article.update` | Update a Draft article |
| 4 | POST | `/articles/v1/search` | `api.article.search` | Search articles with filters |
| 5 | POST | `/articles/v1/review/{articleId}` | `api.article.review` | Submit Draft for review |
| 6 | POST | `/articles/v1/publish/{articleId}` | `api.article.publish` | Publish Review → Live |
| 7 | POST | `/articles/v1/reject/{articleId}` | `api.article.reject` | Reject Review → Draft |
| 8 | POST | `/articles/v1/retire/{articleId}` | `api.article.retire` | Retire a Live article |

Each transition endpoint is documented using the standard Endpoint Documentation Template from SKILL.md.

---

## Filtering by State

`search` endpoints for workflow-enabled resources almost always need a `status` filter so callers can request just the Live ones (or just Drafts they own). Add `status` to the filters block:

```json
{
  "request": {
    "filters": {
      "status": ["Live"]
    }
  }
}
```

When no `status` filter is supplied, the server's default for a workflow-enabled resource is to return only `Live` records to anonymous callers and only `Live` + the caller's own `Draft`/`Review` records to authenticated callers. State this default explicitly in the search endpoint's documentation.