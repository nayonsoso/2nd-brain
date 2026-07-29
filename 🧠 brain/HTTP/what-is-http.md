#understand #ai-draft

# What is HTTP?

==HTTP== stands for **HyperText Transfer Protocol**.
It is the base protocol for sending data on the web.
When your browser loads a page, it uses HTTP.

---
## How It Works

HTTP is a ==request-response protocol==.

1. The **client** sends a request.
2. The **server** processes it and sends a response.

That is one cycle. The connection is then closed (in HTTP/1.1 by default).

---
## Request Structure

A request has three parts.

**1. Request line** — the method and the path.
```
GET /users/42 HTTP/1.1
```

**2. Headers** — extra info about the request.
```
Host: api.example.com
Authorization: Bearer <token>
Content-Type: application/json
```

**3. Body** — the data you send (only for POST, PUT, PATCH).
```json
{ "name": "Yeongseo" }
```

---
## Response Structure

A response also has three parts.

**1. Status line** — the result code.
```
HTTP/1.1 200 OK
```

**2. Headers** — extra info about the response.
```
Content-Type: application/json
```

**3. Body** — the data the server sends back.
```json
{ "id": 42, "name": "Yeongseo" }
```

---
## HTTP Methods

| Method | Meaning |
|--------|---------|
| GET | Read data |
| POST | Create new data |
| PUT | Replace data fully |
| PATCH | Update part of data |
| DELETE | Remove data |

---
## Common Status Codes

| Code | Meaning |
|------|---------|
| 200 | OK |
| 201 | Created |
| 204 | No content |
| 301 | Moved permanently |
| 400 | Bad request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not found |
| 500 | Server error |

---
## Key Properties of HTTP

- ==Stateless== — the server does not remember past requests. Each request is independent.
- ==Text-based== — requests and responses are readable text (in HTTP/1.1).
- ==Flexible== — you can send any type of data (JSON, HTML, files, etc.).

---
## Related Notes

- [[http-and-rest]] — how REST is built on HTTP
- [[http-vs-https]] — how HTTPS adds security
- [[http-versions]] — HTTP/1.1 vs HTTP/2 vs HTTP/3
- [[http-headers]] — common headers in detail

## Review History

-
