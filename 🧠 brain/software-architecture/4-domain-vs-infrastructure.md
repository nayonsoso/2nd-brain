#essential #ai-draft

# Domain vs Infrastructure

Every system has two very different kinds of code.
One is the business rules. The other is the technical tools.
Knowing the difference is one of the most important ideas in software architecture.

## Domain

==Domain== is the pure business rules of your service.
It lives without any framework or database.
Examples: "a coupon cannot be used twice", "you cannot pay if your balance is too low".
These rules exist because of how the business works, not because of any technology.

---
## Infrastructure

==Infrastructure== is the technical tools that run and support the domain.
Examples: MySQL, Redis, a web framework, REST API, AWS S3.
These tools can change. The business rules usually do not.

---
## Why The Split Matters

This line between domain and infrastructure is the line that every later architecture pattern draws.
Keeping the domain free of infrastructure keeps business rules safe when tools change.
If your domain depends on MySQL, switching to PostgreSQL breaks your business rules.
If your domain is pure, you can swap MySQL for PostgreSQL with no risk to the rules.

---
## The Foundation Of DDD

This "keep the domain pure" idea is the basic foundation of ==Domain-Driven Design==.
You learn it early, before the full DDD notes.
Every DDD pattern builds on this separation.

---
## Related Notes

- [[5-ports-and-adapters]] — how the domain talks to infrastructure
- [[8-hexagonal-architecture]] — domain in the center, infra outside
- [[10-domain-driven-design]] — modeling the domain in depth

## Review History

-
