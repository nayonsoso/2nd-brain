#essential #ai-draft

# Tradeoffs and Choosing

Every architecture style has a price.
Understanding the ==tradeoffs== helps you choose the right tool.
Do not pick an architecture because it looks good. Pick it because it fits your problem.

## The Benefits

The biggest ==benefits== of Clean, Hexagonal, and DDD style are testability and flexibility.
Unit testing is easy because business logic has no dependency on the framework or DB.
You can swap the framework or DB with very little change to the business code.
The business logic is clearly protected inside its own boundary.

---
## The Costs

The main ==costs== are complexity and boilerplate.
You end up with many more files and classes than in a simple app.
Even a simple CRUD feature must pass through several layers.
There is a steep learning curve for new developers joining the project.

---
## When NOT To Use

Do not force full Clean or Hexagonal on every project.
Small CRUD apps do not need this level of structure.
Thin domains with very few business rules do not benefit from DDD.
Short-lived or throwaway projects are not worth the overhead.
Match the architecture to the ==complexity of the domain==.

---
## The Main Lesson

No architecture is always right.
Do not memorize diagrams. Understand the price you pay.
Choose on purpose. Know what you gain and what you give up.

---
## Related Notes

- [[8-layered-architecture]] — the cheaper default
- [[9-hexagonal-architecture]] — cost vs isolation
- [[10-clean-architecture]] — cost vs strictness
- [[11-domain-driven-design]] — worth it only for rich domains

## Review History

-
