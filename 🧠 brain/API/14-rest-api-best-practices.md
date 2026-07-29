#essential #ai-draft

# REST API Best Practices

This note is about ==how to design a good REST API==.
It is a companion to [[3-rest-api]], which covers what REST is.

---
## 1. Use Clear and Consistent Naming

Use names that **make sense and follow one style** everywhere.

> Why we should keep 'one style'?
> I heard this from my friend (폴라), her team experienced incident because of updating 'user-profile' but ingnored 'user_profile'

Bad: `GET /getUser`, `POST /createNewUserRecord`
Good: `GET /users/{id}`, `POST /users`

Rules:
- Use **nouns**, not verbs — the HTTP method is already the verb.
- Use **plural** for collections: `/users`, not `/user`
- Use **lowercase** and **hyphens**: `/user-profiles`, not `/userProfiles`

---
## 2. Give Helpful Error Messages

Do not just return a code. Tell the caller what went wrong.

Bad:
```json
{ "error": "error" }
```

Good:
```json
{
  "error": "INVALID_EMAIL",
  "message": "The email field must be a valid email address."
}
```

Use a ==consistent error format== across all endpoints.

---
## 3. Version Your API

When you change the API, old clients may break.
Use versioning so old users still work.

```
/v1/users
/v2/users
```

Never remove or change an old version without warning users first.

---
## 4. Keep Responses Simple

Return ==only what the caller needs==.
Do not return large objects with many unused fields.
If the caller needs more, let them request it.

---
## 5. Be Predictable

Same patterns everywhere. No surprises.

- If `GET /users` returns a list, `GET /orders` should too.
- If one endpoint uses `snake_case` in JSON, all endpoints should.
- If one endpoint paginates, use the same pagination format everywhere.

---
## 6. Handle Pagination Consistently

When a list can be large, always paginate.
Pick one style and use it everywhere.

```
GET /users?page=1&limit=20       ← offset-based
GET /users?cursor=abc123         ← cursor-based
```

Response format:
```json
{
  "data": [...],
  "total": 200,
  "page": 1,
  "limit": 20
}
```

---
## 7. Use Idempotent Methods Correctly

==Idempotent== means: calling it once or ten times gives the same result.

| Method | Idempotent? |
|--------|-------------|
| GET | Yes |
| PUT | Yes |
| DELETE | Yes |
| POST | No |
| PATCH | Not always |

Design PUT so it is always safe to retry.
Be careful with POST — it creates a new resource every time.

---
## 8. Document Everything

A good API has clear docs. Show real examples of requests and responses.
Without docs, even a perfect API is hard to use.

---
## Related Notes

- [[3-rest-api]] — what REST is and how it works
- [[1-what-is-api]] — what an API is
- [[2-types-of-api]] — other API types

## Review History

-
