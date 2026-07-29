#understand #ai-draft

# HTTP Headers

==Headers== are key-value pairs sent with every HTTP request and response.
They carry extra information about the message — who sent it, what format the data is, how to cache it, and more.

---
## Request Headers

These are sent by the client to the server.

| Header | Purpose | Example |
|---|---|---|
| `Host` | The server address | `api.example.com` |
| `Authorization` | Auth token or credentials | `Bearer <jwt-token>` |
| `Content-Type` | Format of the request body | `application/json` |
| `Accept` | Format the client wants back | `application/json` |
| `User-Agent` | Info about the client | `Mozilla/5.0 ...` |
| `Cookie` | Session data | `session=abc123` |

---
## Response Headers

These are sent by the server back to the client.

| Header | Purpose | Example |
|---|---|---|
| `Content-Type` | Format of the response body | `application/json` |
| `Set-Cookie` | Tell the client to store a cookie | `session=abc123; HttpOnly` |
| `Cache-Control` | How long to cache the response | `max-age=3600` |
| `Location` | Redirect URL (used with 301, 302, 201) | `/users/42` |
| `WWW-Authenticate` | Tell the client how to authenticate | `Bearer realm="api"` |

---
## Important Headers for BE Developers

### Authorization
Used to pass auth credentials.
The most common pattern is ==Bearer token==:
```
Authorization: Bearer <jwt-token>
```

### Content-Type
Tells the server (or client) what format the body is in.
```
Content-Type: application/json
Content-Type: multipart/form-data
```
Always set this when sending a body.

### Cache-Control
Controls how the response is cached.
```
Cache-Control: no-cache       ← always revalidate
Cache-Control: max-age=3600   ← cache for 1 hour
Cache-Control: no-store       ← do not cache at all
```

### CORS Headers
Used when a browser makes a request to a different domain.
The server must allow it explicitly.
```
Access-Control-Allow-Origin: https://yourapp.com
Access-Control-Allow-Methods: GET, POST, DELETE
```

---
## Custom Headers

You can define your own headers. By convention, custom headers used to start with `X-`.
```
X-Request-ID: abc-123
X-Api-Version: 2
```
The `X-` prefix is now deprecated but still widely used.

---
## Related Notes

- [[what-is-http]] — HTTP basics
- [[http-and-rest]] — how REST uses headers for auth and content type
- [[http-vs-https]] — HTTPS encrypts headers too

## Review History

-
