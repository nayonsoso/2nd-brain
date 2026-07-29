#understand #ai-draft

# HTTP Status Codes — 3xx Redirects

A `3xx` code means the client needs to go somewhere else to complete the request.

## Code Reference

| Code                     | Meaning                         | When to Use                                                                                     |
| ------------------------ | ------------------------------- | ----------------------------------------------------------------------------------------------- |
| `301 Moved Permanently`  | New permanent URL               | Renamed endpoints during API **evolution**                                                      |
| `302 Found`              | Temporary redirect              | Point somewhere else just for now                                                               |
| `303 See Other`          | Redirect to result using GET    | After POST/DELETE to show the result; at end of async job to send the client to the finished result |
| `304 Not Modified`       | Cached version is still fresh   | Response to conditional GET with `If-None-Match`                                                |
| `307 Temporary Redirect` | Temporary, keep method and body | Redirect a POST without changing it to GET                                                      |
| `308 Permanent Redirect` | Permanent, keep method and body | Permanent move that must keep POST/PUT and body                                                 |

A redirect means: =="Don't use this URL. Go here instead."==
The server puts the new address in a `Location` header.
The client then makes a **second request** to that address.
Most redirects are followed ==automatically== by the browser or client, so you rarely see them.
```
You    → GET http://oldsite.com
Server → 301  Location: https://newsite.com   ← you never see this
Browser→ GET https://newsite.com  (automatic)
Server → 200 OK                              ← this is all you notice
```

---
## Real Examples You Have Already Hit

`301` — HTTP → HTTPS. You type `http://github.com`, you land on `https://github.com`.
A renamed site, like `twitter.com` → `x.com`, works the same way.
The browser ==remembers== a 301 and skips the old URL next time.
```
GET http://github.com  →  301  Location: https://github.com
```

`302` — "log in first." You open `/account` while logged out and get sent to `/login`.
`/account` did not move forever — it is ==temporary==, so after login it works normally.
The browser does ==not remember== a 302; it asks the server again next time.
```
GET /account  (not logged in)  →  302  Location: /login
```

`304` — reload a page. Your browser already has `logo.png` and asks if it changed.
The server says no, so the image is ==not downloaded again== and the page loads fast.
```
GET /logo.png
If-None-Match: "abc123"  →  304 Not Modified   (reuse cached copy)
```

**What is `If-None-Match`? What is `"abc123"` here?**
`If-None-Match` is a request header used for ==conditional requests==.
When your browser first downloads a resource, the server may include an `ETag` header — a unique value (usually a hash) representing this exact version of the resource.
```
GET /logo.png
← 200 OK  ETag: "abc123"    ← server stamps the resource with a version tag
```
On the next request, your browser sends that tag back:
```
GET /logo.png
If-None-Match: "abc123"     ← "reply only if the resource no longer matches this tag"
```
The server compares the tag to the current version.
If unchanged, the ETag is still `"abc123"` — server replies `304 Not Modified`, nothing re-downloaded.
If changed, the ETag is now different (e.g. `"xyz789"`) — server sends the full resource with `200 OK` and the new ETag.
Think of an ETag as a ==version stamp==. `"abc123"` in the example is a placeholder for whatever hash the server actually assigns.

`307` / `308` — same idea as `302` / `301`, but they ==keep the original method and body==.
APIs use these when moving an endpoint that receives data, so the body is not lost.

**What does "keep the method and body" mean?**
When you make a POST request, you send data in the ==request body== — for example, a JSON payload.
```
POST /old-endpoint
Content-Type: application/json
{ "name": "Alice", "email": "alice@example.com" }
```
If the server responds with a redirect, the client repeats the request at the new URL.
With `301` or `302`, old HTTP clients were allowed to switch the method to GET — dropping the body entirely.
```
POST /old-endpoint  { "name": "Alice", ... }
→ 302 Found  Location: /new-endpoint
→ Client retries: GET /new-endpoint    ← body is gone, data is lost
```
`307` and `308` require the client to repeat the ==exact same method and body==.
```
POST /old-endpoint  { "name": "Alice", ... }
→ 307 Found  Location: /new-endpoint
→ Client retries: POST /new-endpoint  { "name": "Alice", ... }    ← body preserved
```
Use `307`/`308` for any endpoint that receives data (POST, PUT, PATCH) so the payload survives the redirect.

---
## Permanent vs. Temporary

```
301 / 308 → "This URL changed forever. Update your links."
302 / 307 → "Use this other URL for now. Keep calling the original."
```

---
## Does the Method Stay the Same?

This is the ==main difference== between the old and new codes.
```
# Old codes (301, 302) MAY switch the method to GET
POST /old-endpoint   (with body)
→ 301 or 302 → client may retry as GET /new-endpoint   ← body is lost

# New codes (307, 308) KEEP the original method and body
POST /old-endpoint   (with body)
→ 307 or 308 → client retries as POST /new-endpoint    ← body is kept
```

**What does "MAY switch" mean? Can the client keep the original method?**
"May switch" means the HTTP spec for `301` and `302` allowed — but did not require — clients to change the method to GET.
In practice, most browsers and HTTP clients ==do== switch to GET (to avoid accidentally re-submitting form data).
But this behavior was never strictly enforced, so edge cases exist:
- Most browsers always convert to GET
- Some strict HTTP clients or server-side libraries may keep the original method
- You cannot predict which behavior a given client will use

The result: with `301`/`302`, you cannot reliably predict whether the body will survive the redirect.
`307` and `308` were created to remove this ambiguity — they ==require== the client to keep the original method and body, with no exceptions.
If keeping the method matters, always use `307`/`308`.

---
## Quick Map

|                          | Permanent | Temporary |
| ------------------------ | --------- | --------- |
| Method may change to GET | `301`     | `302`     |
| Method is preserved      | `308`     | `307`     |

---
## Related Notes

- [[4-http-status-codes]] — overview and decision tree
- [[5-http-2xx-status-codes]] — Success
- [[7-http-4xx-status-codes]] — Client Errors
- [[8-http-5xx-status-codes]] — Server Errors
- [[3-rest-api]] — HTTP methods and their expected status codes

## Review History

- 2026-07-22
