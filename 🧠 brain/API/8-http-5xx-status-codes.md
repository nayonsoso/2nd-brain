#essential #ai-draft

# HTTP Status Codes — 5xx Server Errors

Something went wrong on the server. The client's request might be valid.

## Code Reference

| Code                        | Meaning             | When to Use                             |
| --------------------------- | ------------------- | --------------------------------------- |
| `500 Internal Server Error` | Unexpected failure  | Unhandled exception, DB connection lost |
| `502 Bad Gateway`           | Upstream failure    | External service is down                |
| `503 Service Unavailable`   | Temporarily down    | During deployment or DB maintenance     |
| `504 Gateway Timeout`       | Upstream timed out  | External service didn't respond in time |

---
## Never Return 200 with an Error Body

==Never return `200 OK` with an error body.==
```
❌ 200 OK  { "success": false, "error": "experiment not found" }
✅ 404 Not Found  { "error": "experiment not found" }
```

---
## Related Notes

- [[4-http-status-codes]] — overview and decision tree
- [[5-http-2xx-status-codes]] — Success
- [[6-http-3xx-status-codes]] — Redirects
- [[7-http-4xx-status-codes]] — Client Errors
- [[3-rest-api]] — HTTP methods and their expected status codes
- [[9-idempotency-and-safety]] — how status codes relate to retries

## Review History

-
