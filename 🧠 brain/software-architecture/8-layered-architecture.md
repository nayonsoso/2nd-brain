#understand #ai-draft

# Layered Architecture

==Layered architecture== splits a system into horizontal layers.
Each layer has one job and talks only to the layer below it.

## The Structure

The classic design has three layers.

```
[ Presentation ]   handles HTTP requests/responses
       |
[   Business   ]   holds the business logic
       |
[ Data Access  ]   talks to the database
```

Each layer depends only on the one directly below it.
The top layer handles the user. The bottom layer handles the data.

---
## Why It Is Common

This is the most intuitive layout for most developers.
The layers match how people think about a backend system.
It is easy to understand and very widely used.

---
## The Trap

Business logic easily leaks into the ==data layer==.
When this happens, the two layers become tightly coupled.
If the database changes, the change ripples up into the business code.
This is the main weakness of layered architecture.

---
## The Fix Comes Later

==Hexagonal architecture== and ==Clean architecture== fix this trap.
They invert the dependency on the data layer.
The business code no longer depends on the database.
The database depends on the business code instead.

---
## Related Notes

- [[1-core-design-principles]] — the separation this pattern applies
- [[9-hexagonal-architecture]] — the fix for the coupling trap
- [[10-clean-architecture]] — a stricter fix
- [[13-tradeoffs-and-choosing]] — when layered is enough

## Review History

- 2026-08-04
