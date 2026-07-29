#essential #ai-draft

# HTTP Status Codes — 4xx Client Errors

The ==client did something wrong==. It must fix the request before retrying.

## Code Reference

| Code                       | Meaning                         | When to Use                                                  |
| -------------------------- | ------------------------------- | ------------------------------------------------------------ |
| `400 Bad Request`          | Malformed request               | Unparseable JSON, wrong Content-Type, missing required field |
| `401 Unauthorized`         | Not authenticated               | Missing or expired token                                     |
| `403 Forbidden`            | Not authorized                  | User lacks permission for this resource                      |
| `404 Not Found`            | Resource doesn't exist          | ID not in database                                           |
| `409 Conflict`             | State conflict                  | Duplicate creation, invalid state transition                 |
| `422 Unprocessable Entity` | Valid syntax, invalid semantics | Business rule violations, validation errors                  |
| `429 Too Many Requests`    | Rate limit exceeded             | Include `Retry-After` header                                 |

---
## 400 vs. 422 — Key Distinction

```
400 Bad Request:
  - Malformed JSON
  - Wrong type: { "price": "not-a-number" }
  → "I can't parse what you sent"

422 Unprocessable Entity:
  - End date before start date
  - Price below minimum threshold
  → "I understand it, but it violates business rules"
```

---
## 404 vs. 403 — Security Consideration

```
# Use 404 when resource existence itself is sensitive
GET /experiments/secret-123
→ 404 Not Found   ← hides that the resource exists

# Use 403 when user already knows the resource exists
GET /experiments/123
→ 403 Forbidden   ← user can see it in a list, but can't edit it
```

---
## Related Notes

- [[4-http-status-codes]] — overview and decision tree
- [[5-http-2xx-status-codes]] — Success
- [[6-http-3xx-status-codes]] — Redirects
- [[8-http-5xx-status-codes]] — Server Errors
- [[3-rest-api]] — HTTP methods and their expected status codes
- [[10-request-response-design]] — how to structure error response bodies

## Review History

-
