#essential #ai-draft

# HTTP Status Codes — 2xx Success

A `2xx` code means the server received and processed the request successfully.

## Code Reference

| Code             | Meaning          | When to Use                                   | Body?                                                           |
| ---------------- | ---------------- | --------------------------------------------- | --------------------------------------------------------------- |
| `200 OK`         | Success          | GET; PATCH/PUT returning updated resource     | Yes                                                             |
| `201 Created`    | Resource created | POST that created a resource                  | Yes + `Location` header                                         |
| `202 Accepted`   | Async started    | Request queued, not yet processed             | Optional — status URL to poll / or `Location` header            |
| `204 No Content` | Success, no body | DELETE; PUT/PATCH when not returning resource | No                                                              |

---
## 202 + Polling — Which Code While the Job Runs?

Use `202` only on the request that ==starts== the async job.
Polling is a ==read== of the job's status, so a successful read is `200` — not `202`.
The job's "not done yet" state lives in the body, not in the status code.
```
# 1. Start the job (trigger) — 202 here only
POST /reports
→ 202 Accepted
   Location: /jobs/abc123          ← the status URL to poll

# 2. Poll while running — 200
GET /jobs/abc123
→ 200 OK  { "status": "processing", "progress": 45 }

# 3. Poll after it finishes — 200
GET /jobs/abc123
→ 200 OK  { "status": "completed", "result_url": "/reports/xyz" }
```
Why `200`? The status resource ==already exists== and the server returned it successfully.
`202` means "I accepted your **work request**", and polling is not a new work request.

Optional: on completion you can send `303 See Other` to point straight to the result.
```
GET /jobs/abc123   (completed)
→ 303 See Other
   Location: /reports/xyz          ← client then GETs the finished result
```

---
## Related Notes

- [[4-http-status-codes]] — overview and decision tree
- [[6-http-3xx-status-codes]] — Redirects
- [[7-http-4xx-status-codes]] — Client Errors
- [[8-http-5xx-status-codes]] — Server Errors
- [[3-rest-api]] — HTTP methods and their expected status codes

## Review History

- 2026-07-22
