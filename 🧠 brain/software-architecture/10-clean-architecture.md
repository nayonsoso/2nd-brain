#understand #ai-draft

# Clean Architecture

==Clean Architecture== is a design pattern by Robert C. Martin (Uncle Bob).
It organizes code into concentric circles.
Each circle has a specific role.
The goal is to keep business logic independent of tools and frameworks.

## The Circles

Clean Architecture is drawn as four concentric circles: ==Entities → Use Cases → Interface Adapters → Frameworks==.
Entities are the center.
Frameworks are the outer ring.

```
+-----------------------------+
|   Frameworks & Drivers      |
|  +-----------------------+  |
|  | Interface Adapters    |  |
|  |  +-----------------+  |  |
|  |  |   Use Cases     |  |  |
|  |  |  +-----------+  |  |  |
|  |  |  | Entities  |  |  |  |
|  |  |  +-----------+  |  |  |
|  |  +-----------------+  |  |
|  +-----------------------+  |
+-----------------------------+
```

- ==Entities== — core business rules and data. They change the least.
- ==Use Cases== — application-specific logic. They orchestrate entities.
- ==Interface Adapters== — convert data between use cases and the outside world (e.g. controllers, presenters).
- ==Frameworks & Drivers== — databases, web frameworks, UI. They change the most.

---
## The Dependency Rule

The core rule is: ==source code dependencies point only inward==.
An inner circle never knows anything about an outer circle.
Entities never import a database or a framework.
Use Cases never import a web controller.
Only outer circles depend on inner circles.

---
## Relation To Hexagonal

==Hexagonal Architecture== and Clean Architecture share the same core idea.
Both separate business logic from external tools.
Both use a boundary to protect the inside from the outside.
Clean Architecture generalizes this idea into explicit, named layers.
Hexagonal uses the terms ports and adapters instead.

---
## DIP Enforces It

In practice, you follow the dependency rule by applying ==DIP== (Dependency Inversion Principle).
Inner layers define interfaces.
Outer layers implement those interfaces.
This way, the inner circle never depends on a concrete outer class.
It only depends on an abstraction it owns.

---
## Related Notes

- [[9-hexagonal-architecture]] — the close cousin
- [[6-use-case]] — what the "Use Cases" circle is
- [[2-solid-principles]] — DIP enforces the dependency rule
- [[12-ddd-building-blocks]] — what lives in the entity core
- [[13-tradeoffs-and-choosing]] — the cost of this strictness

## Review History

- 2026-08-04
