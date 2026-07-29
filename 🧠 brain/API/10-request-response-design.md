#essential #ai-draft

# Request/Response Design

## Request Validation

Validate in layers — each layer catches different problems.

```
Layer 1 — Transport:   Content-Type, authentication, request size  → 400/401/413
Layer 2 — Syntax:      JSON parsing, required fields, type check   → 400
Layer 3 — Semantics:   Business rules, cross-field validation      → 422
Layer 4 — State:       Does current state allow this operation?    → 409
```

==Report ALL validation errors at once== — don't make clients fix one error at a time.

```json
// ❌ One error at a time
{ "error": "name is required" }

// ✅ All errors at once
{
  "error": "validation failed",
  "details": [
    { "field": "name", "message": "is required" },
    { "field": "end_date", "message": "must be after start_date" },
    { "field": "market_id", "message": "market 999 does not exist" }
  ]
}
```

---
## Pagination

**Offset/Limit (Page-Based):**
```
GET /experiments?page=3&page_size=20

{
  "data": [...],
  "pagination": {
    "page": 3, "page_size": 20,
    "total_count": 157, "total_pages": 8
  }
}
```
Pros: Simple; can jump to any page.
Cons: ==Performance degrades== with large offsets; inconsistent when data changes.

**Cursor-Based:**
```
GET /experiments?limit=20&cursor=eyJpZCI6MTIzfQ==

{
  "data": [...],
  "pagination": { "next_cursor": "eyJpZCI6MTQzfQ==", "has_more": true }
}
```
Pros: Consistent performance; stable results even as data changes.
Cons: Can't jump to page N; cursor is opaque.

**When to use which:**

| Scenario | Recommendation |
| --- | --- |
| Admin UI with page numbers | Offset/limit |
| Feed / timeline scrolling | Cursor-based |
| Large datasets (100K+ rows) | Cursor-based |
| Export / sync operations | Cursor-based |
| Small, rarely-changing data | Either works |

---
## Filtering & Sorting

**Query parameter conventions:**
```
# Equality filter
GET /experiments?status=active&market_id=42

# Multiple values (OR)
GET /experiments?status=active,paused

# Range filters
GET /experiments?created_after=2024-01-01&created_before=2024-12-31

# Sorting (- prefix = descending)
GET /experiments?sort=-created_at
GET /experiments?sort=-created_at,name
```

**Design principles:**
- Use snake_case for parameter names
- Support sensible defaults — omitting `sort` should default to something useful
- Validate filter values — unknown fields return `400`, not silent ignore
- Document which fields are filterable — not every field should be

---
## Response Structure

==Consistency is king.== Every response should follow the same shape.

```json
// Single resource
{
  "id": "exp-123",
  "name": "Q4 Pricing Test",
  "status": "created",
  "created_at": "2024-03-15T10:30:00Z"
}

// Collection
{
  "data": [{ "id": "exp-123" }, { "id": "exp-124" }],
  "pagination": { "page": 1, "page_size": 20, "total_count": 47 }
}
```

**Nested vs. Referenced:**
```json
// Embedded — client almost always needs this data
{ "id": "exp-123", "market": { "id": 42, "name": "Germany" } }

// Referenced — client rarely needs this data
{ "id": "exp-123", "market_id": 42 }

// Hybrid — best of both
{ "id": "exp-123", "market": { "id": 42, "name": "Germany" } }
```

**Guidelines:**
- Embed data the client almost always needs
- Reference data the client rarely needs
- Never embed unbounded lists — paginate them as sub-resources
- ==Always use ISO 8601 in UTC== for dates: `"2024-03-15T10:30:00Z"`

---
## Standardized Error Responses

Pick a format and use it ==everywhere.==

```json
{
  "error": {
    "code": "EXPERIMENT_STATE_INVALID",
    "message": "Cannot submit experiment in 'archived' state",
    "details": [
      {
        "field": "status",
        "message": "current status 'archived' does not allow transition to 'submitted'",
        "allowed_transitions": ["created"]
      }
    ],
    "request_id": "req-abc-123"
  }
}
```

What makes error responses useful:
- **Machine-readable code** — clients switch on this, not the message
- **Human-readable message** — developers read this in logs
- **Field-level details** — for form validation UIs
- **Request ID** — for correlating with server logs
- **No stack traces in production** — security risk; log them server-side

---
## Related Notes

- [[3-rest-api]] — resource design and URI structure
- [[4-http-status-codes]] — which status codes to use for each error type
- [[9-idempotency-and-safety]] — how state validation prevents bad retries
- [[13-api-performance]] — caching headers, partial responses, batch endpoints
- [[15-api-design-examples]] — real-world examples of these patterns

## Review History

-
