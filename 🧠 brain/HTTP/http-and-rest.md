#understand #ai-draft

# HTTP and REST

==REST is not a protocol. HTTP is.==
REST is a set of rules (an architectural style) that runs *on top of* HTTP.

---
## What HTTP Provides

HTTP gives REST the tools it needs:

| HTTP Feature                     | How REST Uses It                                     |
| -------------------------------- | ---------------------------------------------------- |
| Methods (GET, POST, PUT, DELETE) | Actions on resources (read, create, replace, delete) |
| URLs                             | Addresses for resources (`/users/42`)                |
| Status codes                     | Results of the action (200, 404, 500)                |
| Headers                          | Auth tokens, content type, caching                   |
| Request body                     | Data to create or update a resource                  |

---
## What REST Adds on Top

HTTP alone has no rules about how to design your API.
REST adds ==conventions== so APIs are consistent and predictable.

REST says:
- A URL should represent a **resource**, not an action.
- Use the HTTP method to express the **action**.
- Each request must be **stateless**.
- Responses should be in a standard format (usually JSON).

Without REST rules, two developers could use HTTP very differently.
REST gives a common pattern everyone can follow.

---
## A Simple Example

Without REST thinking:
```
POST /getUserById       ← using POST to read
POST /deleteUser        ← using POST to delete
```

With REST:
```
GET    /users/42        ← read
DELETE /users/42        ← delete
```

Both use HTTP. Only the second follows REST rules.

---
## Summary

|                   | HTTP            | REST                               |
| ----------------- | --------------- | ---------------------------------- |
| What it is        | A protocol      | An architectural style             |
| Defined by        | RFC standards   | Roy Fielding's dissertation (2000) |
| Provides          | Transport rules | API design rules                   |
| Without the other | REST needs HTTP | HTTP works without REST            |

---
## Related Notes

- [[what-is-http]] — what HTTP is
- [[3-rest-api]] — REST in detail
- [[http-headers]] — headers REST uses for auth and content type
- [[4-http-status-codes]] — complete status code reference

## Review History
- 2026-07-22
