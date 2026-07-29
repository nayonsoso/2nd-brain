#aware #ai-draft

# HTTP Versions

HTTP has changed a lot over time.
Each version made it faster or more efficient.

---
## HTTP/1.1 (1997)

The version most developers learned first.

- One request per connection at a time.
- To send multiple requests, the browser opens ==multiple TCP connections== (usually up to 6).
- Text-based format — easy to read and debug.
- Still widely supported today.

**Problem:** slow. Waiting for one response before sending the next is inefficient.

---
## HTTP/2 (2015)

Built to fix the speed problems of HTTP/1.1.

- ==Multiplexing== — multiple requests and responses travel over one connection at the same time.
- **Header compression** — headers are compressed to save bandwidth.
- **Server push** — server can send resources before the client asks.
- Binary format — faster to parse, but not human-readable.
- Requires HTTPS in practice (browsers only support HTTP/2 over TLS).

**Result:** much faster page loads. Most modern servers and browsers support it.

---
## HTTP/3 (2022)

The newest version. Built on a new transport protocol called ==QUIC== instead of TCP.

- QUIC runs over UDP, not TCP.
- Faster connection setup — less round trips to start.
- Better on bad networks — one lost packet does not block the whole connection.
- Still rolling out. Support is growing but not universal.

---
## Summary

| Version | Year | Key Feature |
|---------|------|-------------|
| HTTP/1.1 | 1997 | Standard request/response, multiple connections |
| HTTP/2 | 2015 | Multiplexing, header compression, binary |
| HTTP/3 | 2022 | QUIC over UDP, faster on unstable networks |

---
## What You Need to Know as a BE Developer

- Most backend frameworks support HTTP/1.1 and HTTP/2 out of the box.
- HTTP/2 is the standard to aim for today.
- HTTP/3 support is handled mostly at the infrastructure level (load balancer, CDN), not the app level.
- You do not usually pick the version manually — your server and client negotiate it automatically.

---
## Related Notes

- [[what-is-http]] — what HTTP is
- [[http-vs-https]] — HTTPS is required for HTTP/2 and HTTP/3

## Review History

-
