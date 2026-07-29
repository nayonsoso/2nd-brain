#understand #ai-draft

# Performance & Scalability

## Request/Response Size

**Compression** — use gzip for responses over a few KB:
```
# Client indicates support:
GET /experiments
Accept-Encoding: gzip

# Server compresses response:
Content-Encoding: gzip
```

**Streaming** for large payloads — don't buffer a 100MB export in memory:
```go
w.Header().Set("Transfer-Encoding", "chunked")
encoder := json.NewEncoder(w)
for item := range items {
    encoder.Encode(item)
    w.(http.Flusher).Flush()
}
```

---
## Caching Headers

```
# Cache-Control — tells clients and proxies how to cache:
Cache-Control: public, max-age=300    — cache for 5 min, anyone can cache
Cache-Control: private, max-age=60   — only the client caches
Cache-Control: no-cache              — must revalidate before using
Cache-Control: no-store              — never cache (sensitive data)

# ETag — fingerprint-based validation:
GET /experiments/123
→ 200 OK, ETag: "a1b2c3d4"

GET /experiments/123
If-None-Match: "a1b2c3d4"
→ 304 Not Modified ← save bandwidth
```

**What to cache and for how long:**

| Resource | Cache Strategy |
| --- | --- |
| Static config / reference data | `public, max-age=3600` |
| User-specific data | `private, max-age=60` |
| Rapidly changing data | `no-cache` + ETag |
| Sensitive data (pricing) | `no-store` |

---
## Partial Responses

Allow clients to request only the fields they need:
```
GET /experiments/123?fields=id,name,status
→ { "id": "123", "name": "Q4 Test", "status": "active" }
```

When this matters: mobile clients on slow connections; dashboards showing lists with basic info.

---
## Batch Endpoints

```
# ❌ Without batch — N+1 requests:
GET /experiments/1
GET /experiments/2
GET /experiments/3

# ✅ With batch endpoint:
GET /experiments?ids=1,2,3
→ { "data": [exp1, exp2, exp3] }

# ✅ Batch create:
POST /experiments/batch
[ { "name": "Exp A" }, { "name": "Exp B" } ]
→ 200 OK
{
  "results": [
    { "status": 201, "data": { "id": "123" } },
    { "status": 422, "error": { "message": "invalid market" } }
  ]
}
```

**Design considerations:**
- Set a maximum batch size (e.g., 100 items) — return `400` if exceeded
- Return per-item results — some may succeed, some may fail
- Use `200 OK` as the overall status even if some items fail
- Apply the same validation to each item as the single-resource endpoint

---
## Transactions for Multi-Step Operations

When multiple resources must be created or deleted atomically, use a transaction.

```go
// All three must succeed or all roll back
transactionErr := transactionManager.NewTransaction(ctx, func(ctxWithTx context.Context) error {
    _, err = periodRepo.Create(ctxWithTx, period)
    if err != nil { return err }

    experiment, err = experimentRepo.Create(ctxWithTx, experiment)
    if err != nil { return err }

    err = history.CreateExperiment(ctxWithTx, experiment.ID)
    if err != nil { return err }

    return nil  // commit
})
// any error → auto-rollback
```

---
## Related Notes

- [[10-request-response-design]] — pagination and filtering design
- [[9-idempotency-and-safety]] — batch operations and idempotency
- [[4-http-status-codes]] — 202 Accepted for async operations

## Review History

-
