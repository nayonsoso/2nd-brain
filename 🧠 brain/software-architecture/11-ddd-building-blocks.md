#understand #ai-draft

# DDD Building Blocks

==Domain-Driven Design== has tactical building blocks.
These are the objects you create in code to model your domain.

## Entity

==Entity== is an object with an identity that lasts over time.
Two entities with the same fields are still different if their IDs differ.
Example: a User with an ID.

---
## Value Object

==Value Object== is an object defined only by its values.
It has no identity and should be immutable.

```go
type Money struct { // value object: compared by value, immutable
    Amount   int
    Currency string
}
```

Two Money values with the same amount and currency are equal.
There is no need for a separate ID.

---
## Aggregate

==Aggregate== is a cluster of related objects treated as one unit.
It has an ==aggregate root==.
The outside world only touches the root.
Example: an Order (root) with its OrderItems.

---
## Repository

==Repository== gives collection-like access to aggregates.
It hides the storage details from the domain.
It is where DDD meets ports and adapters: the repository is a port.

---
## Related Notes

- [[10-domain-driven-design]] — the methodology these serve
- [[5-ports-and-adapters]] — the Repository as a port
- [[4-domain-vs-infrastructure]] — these objects live in the domain

## Review History

-
