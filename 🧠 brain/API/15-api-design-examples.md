#understand #ai-draft

# Real-World API Design Examples

## CRUD Pattern — User Management API

```
# Create
POST /v1/users
{ "email": "jane@example.com", "name": "Jane Smith", "role": "analyst" }
→ 201 Created, Location: /v1/users/usr-456
  { "id": "usr-456", "email": "...", "created_at": "2024-03-15T10:30:00Z" }

# List with filtering and pagination
GET /v1/users?role=analyst&sort=-created_at&page=1&page_size=20
→ 200 OK
  { "data": [...], "pagination": { "page": 1, "total_count": 47 } }

# Get one
GET /v1/users/usr-456
→ 200 OK { "id": "usr-456", ... }

# Update (partial)
PATCH /v1/users/usr-456
{ "role": "admin" }
→ 200 OK { "id": "usr-456", "role": "admin", "updated_at": "..." }

# Delete
DELETE /v1/users/usr-456
→ 204 No Content

# Delete non-existent (idempotent)
DELETE /v1/users/usr-999
→ 204 No Content  ← "it's gone" regardless of whether it existed
```

**Design decisions:**
- Use `PATCH` over `PUT` — clients rarely send the full object
- UUIDs prefixed with `usr-` — readable in logs, prevents cross-service ID collision
- `updated_at` auto-changes — clients can detect concurrent modifications

---
## Search API Design

```
# Simple search — GET with query params (cacheable)
GET /v1/experiments?q=pricing&status=active&market_id=42&sort=-relevance

→ 200 OK
{
  "data": [
    {
      "id": "exp-123",
      "name": "Q4 Pricing Optimization",
      "highlights": { "name": "Q4 <em>Pricing</em> Optimization" }
    }
  ],
  "pagination": { "next_cursor": "...", "has_more": true },
  "facets": {
    "status": { "active": 12, "paused": 3 },
    "market": { "Germany": 10, "France": 7 }
  }
}

# Complex search — POST (when query exceeds URL length)
POST /v1/experiments/search
{
  "query": "pricing optimization",
  "filters": {
    "status": ["active", "paused"],
    "market_id": [42, 43],
    "created_after": "2024-01-01"
  },
  "sort": [{ "field": "relevance", "order": "desc" }],
  "cursor": "eyJzY29yZSI6MC44NX0="
}
```

**Key decisions:**
- ==Cursor pagination== for search — results can change between pages
- Facets — aggregate counts for filter values; lets UI show "Active (12)" checkboxes
- POST for complex queries — URL length limits (~2048 chars) make GET impractical

---
## Concurrent Update Conflict (Optimistic Locking)

**Problem**: Two users edit the same resource. Second save silently overwrites the first.

**Solution**: ETags + `If-Match` header.

```
# User A fetches
GET /v1/experiments/exp-123
→ 200 OK, ETag: "version-7"

# User B fetches
GET /v1/experiments/exp-123
→ 200 OK, ETag: "version-7"

# User A updates — succeeds
PATCH /v1/experiments/exp-123
If-Match: "version-7"
{ "name": "User A's Name" }
→ 200 OK, ETag: "version-8"

# User B updates — fails (stale version)
PATCH /v1/experiments/exp-123
If-Match: "version-7"
{ "name": "User B's Name" }
→ 409 Conflict
{
  "error": {
    "code": "CONCURRENT_MODIFICATION",
    "message": "Resource was modified. Fetch the latest version and retry.",
    "current_etag": "version-8"
  }
}
```

User B must re-fetch, see User A's changes, then retry with the new ETag.

---
## Error Responses — Good vs. Bad

```json
// ❌ Bad: vague
{ "error": "Bad request" }

// ❌ Bad: exposes internals
{ "error": "pq: duplicate key value violates unique constraint" }

// ❌ Bad: wrong status code
// HTTP 200 OK
{ "success": false, "error": "Experiment not found" }

// ✅ Good: specific, actionable, safe
// HTTP 409 Conflict
{
  "error": {
    "code": "DUPLICATE_EXPERIMENT",
    "message": "An experiment with name 'Q4 Test' already exists in market 'Germany'",
    "details": [
      {
        "field": "name",
        "message": "must be unique within a market",
        "conflicting_resource": "/v1/experiments/exp-789"
      }
    ],
    "request_id": "req-abc123"
  }
}
```

**Error response principles:**
1. Errors are for two audiences: machines (error codes) and humans (messages)
2. Include enough context to debug without accessing server logs
3. Never expose internals (SQL queries, stack traces, file paths)

---
## Related Notes

- [[3-rest-api]] — resource design and HTTP methods
- [[4-http-status-codes]] — which status code to use when
- [[9-idempotency-and-safety]] — idempotent deletes, optimistic locking
- [[10-request-response-design]] — pagination, filtering, error response structure
- [[13-api-performance]] — batch endpoints design

## Review History

-
