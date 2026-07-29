#aware #ai-draft

# Code Review Assignments

Practice exercises for API design and implementation.

## Assignment 1: Review a Poorly-Designed API

Find the issues in this API. For each issue, explain what's wrong and propose a fix.

```
POST /api/getAllUsers
Body: { "page": 1 }
→ 200 OK  { "success": true, "data": [...], "error": null }

POST /api/getUser
Body: { "userId": "123" }
→ 200 OK  { "success": true, "data": { "user_Id": "123", "Name": "Jane" } }

POST /api/getUser
Body: { "userId": "999" }
→ 200 OK  { "success": false, "data": null, "error": "User not found" }

POST /api/createUser
Body: { "Name": "Jane", "email": "jane@test.com" }
→ 200 OK  { "success": true, "data": { "user_Id": "456" } }
```

Expected findings:
1. All POST — use GET for reads, DELETE for deletes, PATCH for updates
2. RPC-style URLs — `/api/getAllUsers` → `GET /v1/users`
3. Always 200 — use HTTP status codes, not a `success` boolean
4. Inconsistent casing — `user_Id`, `Name` → pick snake_case
5. No versioning — `/api/` should be `/v1/`
6. No pagination metadata — no page info, no links
7. No `Location` header on create
8. IDs in body instead of URL — `GET /v1/users/123`
9. Create returns only ID — return the full resource
10. No error codes — just a string message; clients can't switch on it
11. Search not cacheable — simple searches should be GET

---
## Assignment 2: Design an API for Pricing Rules

**Scenario**: Design an API for managing "Pricing Rules". A rule:
- Belongs to a market
- Has a name, description, type (percentage, absolute, formula), and value
- Can be active or inactive
- Has a priority (integer, lower = higher priority)
- Can apply to specific item categories (many-to-many)

**Deliverables:**
1. List all endpoints with HTTP methods, paths, request/response bodies
2. Define error scenarios and status codes for each endpoint
3. Design filtering/sorting for the list endpoint
4. Handle the case where two rules have conflicting priorities

**Starter template:**
```
Base path: /v1/markets/{market_id}/pricing-rules

1. Create     POST   /v1/markets/{market_id}/pricing-rules
2. List       GET    /v1/markets/{market_id}/pricing-rules?...
3. Get one    GET    /v1/markets/{market_id}/pricing-rules/{rule_id}
4. Update     PATCH  /v1/markets/{market_id}/pricing-rules/{rule_id}
5. Delete     DELETE /v1/markets/{market_id}/pricing-rules/{rule_id}
6. Reorder    PUT    /v1/markets/{market_id}/pricing-rules/order
7. Categories PUT    /v1/markets/{market_id}/pricing-rules/{rule_id}/categories
8. Toggle     PATCH  /v1/markets/{market_id}/pricing-rules/{rule_id}
```

---
## Assignment 3: Implement One Endpoint

**Task**: Implement a "Create Pricing Rule" endpoint in Go.

**Requirements:**
- Validate all inputs (name required, valid type, positive priority)
- Return proper errors for each failure case
- Handle duplicate name within a market (409 Conflict)
- Use the transaction manager pattern
- Include `Location` header in the response

```go
func (s *Service) Create(ctx context.Context, marketID uuid.UUID, input CreateInput) (*entity.PricingRule, error) {
    // 1. Validate input
    //    name required, type must be percentage|absolute|formula, priority > 0

    // 2. Check market exists
    //    not found → return not found error

    // 3. Check for duplicate name in market
    //    duplicate → return conflict error

    // 4. Create within transaction
    //    create rule, assign categories if provided

    // 5. Return created rule
}
```

---
## Assignment 4: Write OpenAPI Documentation

**Task**: Write an OpenAPI 3.0 snippet for `GET /v1/experiments/{id}` and `PATCH /v1/experiments/{id}`.

Include:
- Path parameters
- Query parameters (for partial response: `?fields=...`)
- Request body schema (for PATCH)
- Response schemas for success and all error cases
- Authentication requirement
- Example values
- `If-Match` header for optimistic locking on PATCH
- `409` response for concurrent modification

---
## Assignment 5: Trace a Request Flow

Pick one endpoint and trace the complete flow: DSL → Controller → Service → Repository.

**Option A**: `GET /api/v1/experiment` (List)
- How are pagination defaults enforced?
- What happens if a required param is missing?
- Where does error handling happen vs. where is it mapped to HTTP?

**Option B**: `POST /api/v1/zone/{id}/trigger_segmentation`
- Why is this a POST and not a PATCH?
- What does the client get immediately (async processing)?
- How would you redesign this as resource-oriented?

**Option C**: `DELETE /api/v1/zone/{id}`
- What validation happens before the delete?
- Why is a transaction needed?
- Is this delete idempotent? What happens on the second call?

---
## Assignment 6: Improve Error Handling

**Current state:**
```
CreationError  → 500
WrongData      → 400
NotFound       → 404
ForbiddenError → 403
Everything else → 500
```

**Issues to fix:**
1. `InvalidExperimentState` → `WrongData` → 400. Should it be 409 Conflict instead?
2. `UniqueConstraintViolation` → `WrongData` → 400. Should it be 409 Conflict?
3. No 422 Unprocessable Entity. When should you add one?
4. Generic 500 message has no request ID. How would you add tracing?
5. Design a richer error response struct useful to API consumers.

---
## Related Notes

- [[3-rest-api]] — HTTP methods, resource design, URI structure
- [[4-http-status-codes]] — status codes for each scenario
- [[9-idempotency-and-safety]] — safe retries and idempotency patterns
- [[10-request-response-design]] — validation, pagination, error responses
- [[11-api-versioning]] — breaking changes and deprecation
- [[12-api-authentication-and-security]] — JWT, RBAC, rate limiting
- [[13-api-performance]] — batch operations, caching, transactions
- [[15-api-design-examples]] — real-world design walkthroughs

## Review History

-
