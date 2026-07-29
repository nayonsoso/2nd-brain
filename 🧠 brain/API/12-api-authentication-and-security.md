#essential #ai-draft

# Authentication & Security

## Authentication Methods

**API Keys** — simple, for server-to-server communication:
```
GET /experiments
Authorization: Bearer pt_live_abc123def456
```
Use for: internal services, machine-to-machine.
Don't use for: user-facing APIs — keys are long-lived and hard to scope.

**JWT (JSON Web Tokens)** — stateless, self-contained:
```
GET /experiments
Authorization: Bearer eyJhbGciOiJSUzI1NiIs...

# Decoded payload:
{
  "sub": "user-123",
  "exp": 1711000000,
  "roles": ["pricing-admin"],
  "market_ids": [42, 43]
}
```
Pros: Stateless (no DB lookup per request); contains claims for authorization.
Cons: ==Can't be revoked before expiry== — use short TTL + refresh tokens.

**OAuth2** — for third-party access delegation:
```
# User authorizes → app gets access token → app calls API
GET /experiments
Authorization: Bearer <oauth2_access_token>
```
Use for: third-party integrations, public APIs where users grant specific permissions.

---
## Rate Limiting

**Standard headers:**
```
HTTP/1.1 200 OK
RateLimit-Limit: 100         — max requests per window
RateLimit-Remaining: 42      — requests left in current window
RateLimit-Reset: 1711000060  — Unix timestamp when window resets

# When exceeded:
HTTP/1.1 429 Too Many Requests
Retry-After: 30              — seconds to wait before retrying
```

**Implementation strategies:**

| Strategy | Description | Best For |
| --- | --- | --- |
| Fixed window | 100 req/min, resets on the minute | Simple |
| Sliding window | 100 req in any 60-second span | Prevents boundary burst |
| Token bucket | Tokens regenerate at fixed rate | Allows short bursts |
| Per-endpoint | Different limits per endpoint | Expensive endpoints get lower limits |

---
## CORS

CORS matters when a browser-based client calls your API from a different domain.

```
# 1. Browser sends preflight:
OPTIONS /experiments
Origin: https://app.example.com
Access-Control-Request-Method: POST

# 2. Server responds:
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PATCH, DELETE
Access-Control-Max-Age: 86400   — cache preflight for 24h

# 3. Browser sends actual request (if allowed)
```

**Key rules:**
- ==Never use `Access-Control-Allow-Origin: *` with credentials==
- Whitelist specific origins; don't allow all
- Set `Max-Age` to avoid a preflight on every request
- For server-to-server APIs, CORS is irrelevant — it is browser-only

---
## Input Sanitization

```
# SQL Injection — prevented by parameterized queries
❌ db.Where("name = '" + userInput + "'")
✅ db.Where("name = ?", userInput)

# Size limits — prevent denial-of-service via large payloads
r.Body = http.MaxBytesReader(w, r.Body, 1<<20) // 1MB max
```

---
## RBAC with JWT

Different endpoints require different roles.

```
Write operations  → admin + pricing_manager only
Read operations   → all roles including viewer
```

Authorization flow:
1. Extract claims from JWT token
2. Check application-level access
3. Check required scopes against granted permissions
4. Return `401` if not authenticated, `403` if not authorized

---
## Related Notes

- [[3-rest-api]] — HTTP methods and their safety properties
- [[4-http-status-codes]] — 401 vs 403 explained
- [[11-api-versioning]] — authentication changes as breaking changes

## Review History

-
