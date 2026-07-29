#understand #ai-draft

# API Versioning & Evolution

## Versioning Strategies

**URL Path Versioning** — most common, recommended:
```
GET /v1/experiments/123
GET /v2/experiments/123
```
Pros: Obvious; easy to route; clients see version immediately; works with any HTTP client.
Cons: URL changes; version isn't part of the resource identity.

**Header Versioning:**
```
GET /experiments/123
Accept: application/vnd.pricetool.v2+json
```
Pros: Clean URLs; version is metadata, not identity.
Cons: Harder to test; easy to forget; inconsistent tooling support.

**Query Parameter:**
```
GET /experiments/123?version=2
```
Pros: Easy to add; visible in URLs.
Cons: Mixes concerns; can be cached without the version.

==Recommendation: URL path versioning.==
It is the most explicit and easiest to understand.
Header versioning's theoretical purity doesn't outweigh its practical costs.

---
## What Is a Breaking Change?

| Change | Breaking? | Why |
| --- | --- | --- |
| Add a new optional field to request | No | Old clients don't send it |
| Add a new field to response | No | Clients should ignore unknown fields |
| Add a new endpoint | No | Old clients don't call it |
| Add a new enum value to response | Maybe | Clients with exhaustive switch/case will break |
| Remove a field from response | Yes | Clients relying on it will fail |
| Rename a field | Yes | Same as remove + add |
| Change a field's type | Yes | Parsing will fail |
| Make an optional field required | Yes | Old clients don't send it |
| Change the meaning of a value | Yes | Silent semantic breakage |
| Remove an endpoint | Yes | Clients calling it get 404 |
| Change a success status code | Yes | Clients checking `201` will see `200` |

---
## Deprecation Strategy

```
Phase 1 — Announce (3–6 months before removal):
  - Add Deprecation and Sunset headers
  - Update documentation with migration guide
  - Log usage to identify affected clients

  Deprecation: true
  Sunset: Sat, 01 Mar 2025 00:00:00 GMT
  Link: </v2/experiments>; rel="successor-version"

Phase 2 — Warn (1–3 months before removal):
  - Return Warning header on every response
  - Notify known consumers directly

  Warning: 299 - "Endpoint deprecated. Migrate to /v2/experiments by 2025-03-01"

Phase 3 — Remove:
  - Return 410 Gone (not 404 — it existed, now it's intentionally removed)
  - Include migration instructions in the error body

  410 Gone
  { "error": "Removed on 2025-03-01. Use /v2/experiments instead." }
```

---
## When to Version vs. When to Extend

**Extend** (same version) when changes are backward-compatible:
- Adding optional fields
- Adding new endpoints
- Adding new query parameters
- Relaxing validation (accepting more input)

**Version** when you must make breaking changes:
- Restructuring response format
- Changing authentication mechanism
- Changing field types or meanings
- Fundamental redesign of resource models

==Rule of thumb==: if you can add a default and ignore unknown fields, extend.
If clients would break, version.

---
## Related Notes

- [[3-rest-api]] — resource and URI design principles
- [[14-rest-api-best-practices]] — general API design guidelines

## Review History

-
