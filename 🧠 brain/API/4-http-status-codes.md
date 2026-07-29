#essential #ai-draft

# HTTP Status Codes

Status codes are the API's first communication channel.
The right code tells a client ==what happened== and ==what to do about it== — before reading the body.

## Code Classes

- [[5-http-2xx-status-codes]] — Success
- [[6-http-3xx-status-codes]] — Redirects
- [[7-http-4xx-status-codes]] — Client Errors
- [[8-http-5xx-status-codes]] — Server Errors

---
## Decision Tree

```
Is the request well-formed?
├─ No → 400
└─ Yes → Authenticated?
         ├─ No → 401
         └─ Yes → Authorized?
                  ├─ No → 403 (or 404 to hide existence)
                  └─ Yes → Resource exists?
                           ├─ No → 404
                           └─ Yes → Passes validation?
                                    ├─ No → 422
                                    └─ Yes → State conflict?
                                             ├─ Yes → 409
                                             └─ No → Process
                                                     ├─ Success → 2xx
                                                     └─ Failure → 5xx
```

---
## Related Notes

- [[3-rest-api]] — HTTP methods and their expected status codes
- [[9-idempotency-and-safety]] — how status codes relate to retries
- [[10-request-response-design]] — how to structure error response bodies
- [[what-is-http]] — HTTP basics

## Review History

- 2026-07-28