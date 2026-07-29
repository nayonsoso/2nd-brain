#aware #ai-draft

# HTTP vs HTTPS

==HTTPS== is HTTP with a security layer on top.
The "S" stands for **Secure**.

---
## The Difference

| | HTTP | HTTPS |
|---|---|---|
| Data | Sent as plain text | Encrypted |
| Security | None | TLS encryption |
| Port | 80 | 443 |
| URL | `http://` | `https://` |
| Use today | Rare | Standard |

---
## What TLS Does

HTTPS uses ==TLS== (Transport Layer Security) to:

1. **Encrypt** the data — no one in the middle can read it.
2. **Authenticate** the server — you know you are talking to the real server, not a fake one.
3. **Verify integrity** — the data was not changed during transfer.

---
## Why It Matters for BE Developers

- All production APIs must use HTTPS. Never HTTP in production.
- Browsers block mixed content — if your page is HTTPS, all requests must be HTTPS too.
- Some features (like service workers and geolocation) only work on HTTPS.
- TLS adds a small overhead, but it is negligible in modern systems.

---
## TLS Certificates

To use HTTPS, your server needs a ==TLS certificate==.
The certificate proves your server is who it says it is.

You can get free certificates from **Let's Encrypt**.
Most cloud providers (AWS, GCP, Azure) also handle this for you automatically.

---
## Related Notes

- [[what-is-http]] — what HTTP is
- [[http-versions]] — HTTP/2 and HTTP/3 require HTTPS

## Review History

-
