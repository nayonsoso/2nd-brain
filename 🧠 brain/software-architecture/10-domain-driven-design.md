#understand #ai-draft

# Domain-Driven Design

## A Methodology, Not A Layout

==DDD== is a way to design software, not a file layout.
It is about modeling the business well.
It is different from structural patterns like Layered or Clean.

---
## Ubiquitous Language

==Ubiquitous Language==: business experts and developers use one shared language.
The same words in talk, in code, and in the model.
This removes translation mistakes.

---
## Bounded Context

==Bounded Context==: a clear boundary where one model and its language stay consistent.
The word "order" can mean different things in different contexts.
Each context has its own model.

---
## How It Fits With Clean/Hexagonal

Clean and Hexagonal give you the protected shell (they control dependencies).
DDD fills that core with a rich, well-modeled domain.
They work together.

---
## Related Notes

- [[4-domain-vs-infrastructure]] — the domain idea this builds on
- [[11-ddd-building-blocks]] — the tactical objects of DDD
- [[9-clean-architecture]] — the shell DDD fills

## Review History

-
