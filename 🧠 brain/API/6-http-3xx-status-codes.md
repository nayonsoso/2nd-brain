#understand #ai-draft

# HTTP Status Codes — 3xx Redirects

A `3xx` code means: =="Don't use this URL. Go here instead."==
The server puts the new address in a `Location` header.
The client makes a **second request** to that address.
Most redirects are followed ==automatically== by the browser, so you rarely see them.
```
You    → GET http://oldsite.com
Server → 301  Location: https://newsite.com   ← you never see this
Browser→ GET https://newsite.com  (automatic)
Server → 200 OK                              ← this is all you notice
```

## Code Reference

| Code                     | Meaning                         | When to Use                                                                                         |
| ------------------------ | ------------------------------- | --------------------------------------------------------------------------------------------------- |
| `301 Moved Permanently`  | New permanent URL               | Renamed endpoints during API **evolution**                                                          |
| `302 Found`              | Temporary redirect              | Point somewhere else just for now                                                                   |
| `303 See Other`          | Redirect to result using GET    | After POST/DELETE to show the result; at end of async job to send the client to the finished result |
| `304 Not Modified`       | Cached version is still fresh   | Response to conditional GET with `If-None-Match`                                                    |
| `307 Temporary Redirect` | Temporary, keep method and body | Redirect a POST without changing it to GET                                                          |
| `308 Permanent Redirect` | Permanent, keep method and body | Permanent move that must keep POST/PUT and body                                                     |

---
## 301 / 302 — Permanent vs. Temporary

`301 Moved Permanently` — the URL changed forever.
The browser ==remembers== it and skips the old URL next time.
```
GET http://github.com  →  301  Location: https://github.com
```
Real examples: HTTP → HTTPS upgrade; site renames like `twitter.com` → `x.com`.

`302 Found` — a temporary redirect; the original URL still works later.
The browser does ==not remember== it and checks again next time.
```
GET /account  (not logged in)  →  302  Location: /login
```
Real example: "log in first." After login, `/account` works normally again.

---
## 307 / 308 — Method-Safe Versions of 302 / 301

`307` and `308` are the modern replacements for `302` and `301`.
The only difference: they ==require the client to keep the original method and body==.

**Why does this matter?**
When you POST to an endpoint, your request has a body — for example, a JSON payload.
With `301`/`302`, old clients were **allowed to switch the method to GET** when following the redirect, which drops the body entirely.
```
POST /old-endpoint  { "name": "Alice", ... }
→ 302 Found  Location: /new-endpoint
→ Client retries: GET /new-endpoint    ← body is gone, data is lost
```
`307`/`308` require the client to repeat the ==exact same method and body==.
```
POST /old-endpoint  { "name": "Alice", ... }
→ 307 Found  Location: /new-endpoint
→ Client retries: POST /new-endpoint  { "name": "Alice", ... }    ← body preserved
```
Use `307`/`308` for any endpoint that receives data (POST, PUT, PATCH) so the payload survives the redirect.

**Why did 301/302 allow switching to GET?**
The HTTP spec for `301` and `302` allowed — but did not require — clients to change the method to GET.
In practice, most browsers ==do== switch to GET (to avoid accidentally re-submitting form data).
But this was never strictly enforced, so behavior varies by client — you cannot predict it.
`307`/`308` were created to remove this ambiguity by making method preservation explicit.

---
## Quick Map

|                          | Permanent | Temporary |
| ------------------------ | --------- | --------- |
| Method may change to GET | `301`     | `302`     |
| Method is preserved      | `308`     | `307`     |

```
301 / 308 → "This URL changed forever. Update your links."
302 / 307 → "Use this other URL for now. Keep calling the original."
```

---
## 304 — Not a Redirect, a Cache Signal

`304 Not Modified` looks like a redirect but works differently.
It does not send the client to a new URL.
It tells the client: =="your cached copy is still valid, use it."==

**How it works:**
When your browser first downloads a resource, the server may include an `ETag` — a unique hash representing that exact version.
```
GET /logo.png
← 200 OK  ETag: "abc123"    ← server stamps the resource with a version tag
```
On the next request, the browser sends that tag back in `If-None-Match`:
```
GET /logo.png
If-None-Match: "abc123"     ← "only send the resource if it no longer matches this tag"
← 304 Not Modified          ← tag still matches, reuse cached copy
```
If the resource changed, the ETag is now different (e.g. `"xyz789"`) — server sends the full resource with `200 OK` and the new ETag.

Think of an ETag as a ==version stamp==.
`"abc123"` is a placeholder for whatever hash the server actually assigns.

---
## Related Notes

- [[4-http-status-codes]] — overview and decision tree
- [[5-http-2xx-status-codes]] — Success
- [[7-http-4xx-status-codes]] — Client Errors
- [[8-http-5xx-status-codes]] — Server Errors
- [[3-rest-api]] — HTTP methods and their expected status codes

## Review History

- 2026-07-29
