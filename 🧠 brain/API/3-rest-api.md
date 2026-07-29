#essential #ai-draft

# REST API

## What is REST?

REST stands for ==Representational State Transfer==.
It is a set of **rules** for building APIs over **HTTP**.
An API that follows these rules is called a **RESTful API**.
It is an ==architectural style==, not a protocol.
Its constraints aim for ==scalability, simplicity, modifiability, and reliability==.

---
## Core Ideas

### 1. Resources

Everything is a **resource** — a noun, not a verb.
A resource is a thing: a user, an order, a product.
Each resource has a **URL** (called an endpoint).
```
/users            → all users
/users/42         → user with id 42
/users/42/orders  → orders of user 42
```

Model resources as ==nouns==. If an endpoint name is a verb, ask if that verb is really an HTTP method.
```
❌ POST /getExperimentsByMarket
❌ POST /createNewExperiment
✅ GET  /experiments?market_id=42
✅ POST /experiments
```

Sometimes an action *is* the resource. Model it as a noun:
```
POST /experiments/123/calculations   — "create a calculation"
POST /experiments/123/submissions    — "create a submission"
```

### 2. HTTP Methods

You use HTTP **methods** to do actions on resources.

| Method  | Action                            | ==Idempotent==? | Safe? |
| ------- | --------------------------------- | --------------- | ----- |
| GET     | Read                              | Yes             | Yes   |
| POST    | **Create / trigger action**       | No              | No    |
| PUT     | Replace fully                     | Yes             | No    |
| PATCH   | Update part                       | Usually*        | No    |
| DELETE  | Remove                            | Yes             | No    |
| HEAD    | GET without body                  | Yes             | Yes   |
| OPTIONS | Discover methods / CORS preflight | Yes             | Yes   |

*PATCH is idempotent for "set X to value", not for "increment X".

Common mistakes:
```
❌ POST /experiments/123      — don't use POST for updates
❌ GET  /experiments/delete   — don't encode actions in GET URLs
✅ PATCH /experiments/123     — partial update
✅ DELETE /experiments/123    — clear intent
```

Use **POST beyond creation** when the operation is not idempotent (e.g. triggering a calculation) or does not fit CRUD (e.g. `/experiments/123/submit`).
→ See [[9-idempotency-and-safety]] for safe and idempotent methods in depth.

### 3. Stateless

Each request must have all the info the server needs.
The server ==does not remember the previous request==.
**No session stored on the server side.**
The client sends a token (like **JWT**) with every request.
→ See [[12-api-authentication-and-security]] for auth details.

### 4. JSON

REST APIs almost always use **JSON** for request and response bodies.
```json
{
  "id": 42,
  "name": "Yeongseo",
  "email": "yeongseo@example.com"
}
```

---
## URI Structure

URIs should be ==hierarchical==, showing ownership and relationships.

| Rule                    | Example                              | Why                               |
| ----------------------- | ------------------------------------ | --------------------------------- |
| Use plural nouns        | `/users` not `/user`                 | Collections are plural            |
| Lowercase + hyphens     | `/user-profiles` not `/userProfiles` | Readable; URIs are case-sensitive |
| Nest for ownership      | `/markets/42/experiments`            | Shows relationship                |
| ==Max 2–3 levels deep== | `/markets/42/experiments/123`        | Beyond this, use query params     |
| No trailing slashes     | `/experiments` not `/experiments/`   | Consistency                       |
| No file extensions      | `/experiments/123` not `.json`       | Use the `Accept` header           |

**Nesting vs. Flat:**
```
# Nested — child does not exist without the parent
GET /markets/42/experiments

# Flat with filter — child has its own identity
GET /experiments?market_id=42
GET /experiments/123
```

When a resource has a **globally unique ID** and **clients access it directly**, give it a top-level (flat) endpoint.

---
## Request & Response Structure

Request:
```
METHOD /path HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Content-Type: application/json

{ "body": "here" }
```

Response:
```
HTTP/1.1 200 OK
Content-Type: application/json

{ "response": "here" }
```

→ See [[10-request-response-design]] for validation, pagination, filtering, and error format.

---
## HATEOAS

==HATEOAS (Hypermedia as the Engine of Application State)== means responses include links that **tell the client what to do next.**
```json
{
  "id": "exp-123",
  "status": "created",
  "_links": {
    "self": { "href": "/experiments/exp-123" },
    "submit": { "href": "/experiments/exp-123/submissions", "method": "POST" }
  }
}
```
Full HATEOAS is rare. What is practical: pagination links, self links, and a `Location` header on `201 Created`.

---
## Related Notes

- **Dedicated deep-dives** — sections split out from this note
	- [[4-http-status-codes]] — status codes (200, 404, 500, ...) in detail
	- [[10-request-response-design]] — validation, pagination, filtering, error format
	- [[11-api-versioning]] — how to version and evolve the API
	- [[2-types-of-api]] — REST vs GraphQL vs gRPC vs others
- **Related concepts** — good to read together
	- [[1-what-is-api]] — what an API is
	- [[14-rest-api-best-practices]] — how to design a good API
	- [[http-and-rest]] — how HTTP and REST relate
	- [[what-is-http]] — the protocol REST runs on

## Review History

- 2026-07-22