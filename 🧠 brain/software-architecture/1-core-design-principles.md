#essential #ai-draft

# Core Design Principles

Good software is easy to change.
Architecture is mostly about managing ==dependencies== and ==change==.
When you change one part, you want to avoid breaking other parts.

## Separation of Concerns (SoC)

==Separation of Concerns== means splitting a system into separate modules.
Each module has one clear, non-overlapping job.
For example, do not mix business logic, DB access, and HTTP handling in one class.
Keep them in separate parts of the code.

---
## Loose Coupling

==Loose coupling== means modules have as few connections to each other as possible.
When modules are loosely coupled, you can change one without breaking another.
This makes the system easier to update and test.

---
## High Cohesion

==High cohesion== means related things live together inside one module.
Things that change together should be kept together.
A module with high cohesion has one clear purpose.

---
## The Common Goal

Low coupling and high cohesion is the recurring target of every architecture pattern.
Each pattern you learn is a different way to reach this same goal.

---
## Related Notes

- [[2-solid-principles]] — the object-level rules that support these goals
- [[4-domain-vs-infrastructure]] — the first big separation to make
- [[7-layered-architecture]] — the most common way to separate concerns

## Review History

-
