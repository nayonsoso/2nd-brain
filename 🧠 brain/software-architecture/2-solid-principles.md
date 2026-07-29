#essential #ai-draft

# SOLID Principles

==SOLID== is five object-level rules for writing code that is easy to change.
Each rule reduces a different source of unnecessary coupling.

## Single Responsibility (SRP)

==SRP== says a module should have one reason to change.
One class, one job.
For example, a `UserService` should handle user logic only, not also send emails.

---
## Open/Closed (OCP)

==OCP== says code should be open to extension but closed to modification.
Add new behavior without editing existing code.
For example, add a new payment method by writing a new class, not by changing old ones.

---
## Liskov Substitution (LSP)

==LSP== says a subtype must work anywhere its base type is used, without surprises.
If `Bird` has a `fly()` method, a `Penguin` subclass that throws an error on `fly()` breaks LSP.
Replace the base type with any subtype and the program must still behave correctly.

---
## Interface Segregation (ISP)

==ISP== says many small interfaces are better than one fat interface.
Clients should depend only on what they actually use.
For example, split a large `Repository` interface into `ReadRepository` and `WriteRepository` so read-only clients are not forced to implement write methods.

---
## Dependency Inversion (DIP)

==DIP== says high-level modules (business logic) must not depend on low-level modules (DB, external API).
Both must depend on ==abstractions== (interfaces).
For example, `OrderService` depends on an `OrderRepository` interface, not on a concrete `MySQLOrderRepository`.
This is the key principle that lets Clean Architecture and Hexagonal Architecture flip the direction of dependencies.

---
## DIP vs DI

==DIP== is the *principle*: depend on abstractions, not concretions.
==DI== (dependency injection) is the *technique* that helps you follow it.
DI means passing dependencies in from the outside (via constructor, setter, or framework) instead of creating them inside.
DIP tells you *what* to do. DI is one *way* to do it.

---
## Related Notes

- [[1-core-design-principles]] — the higher-level goals SOLID serves
- [[3-dependency-injection]] — the technique that realizes DIP
- [[9-hexagonal-architecture]] — DIP applied to isolate the core
- [[10-clean-architecture]] — DIP as the dependency rule

## Review History

- 2026-07-29
