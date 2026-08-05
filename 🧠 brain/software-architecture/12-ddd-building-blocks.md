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
This means outside code must never change a child object by hand.
It always asks the root to do it.

```go
// Bad: the caller grabs the child and changes it.
order.Items[0].Quantity = 100

// Good: the caller asks the root.
order.ChangeQuantity(itemID, 100)
```

The reason is simple.
==Only the root knows the rules of the whole aggregate.===
Say an order cannot cost more than the limit, and a shipped order cannot change.
`Items[0].Quantity = 100` checks neither rule.
An OrderItem only knows itself. 
It does not know the total price or the ship status.

```go
type Order struct { // aggregate root
    items  []OrderItem // unexported: nothing outside the package can touch them
    status OrderStatus
}

func (o *Order) ChangeQuantity(itemID int64, quantity int) error {
    if o.status != OrderCreated {
        return errors.New("cannot change a shipped order")
    }
    item, err := o.findItem(itemID)
    if err != nil {
        return err
    }
    item.Quantity = quantity
    if o.TotalPrice() > MaxOrderPrice {
        return errors.New("total price is over the limit")
    }
    return nil
}
```

If you leave a side door open, every change through that door skips these checks.
So you keep ==one door only==, and you put the checks on it.

Three rules follow from this.

**1. Do not hand out your insides.**
A getter that returns the real slice is the same side door.
The caller can still append or edit through it.

```go
// Bad: shares the same backing array.
func (o *Order) Items() []OrderItem { return o.items }

// Good: give back a copy.
func (o *Order) Items() []OrderItem { return slices.Clone(o.items) }
```

**2. One repository per aggregate.**
You write an `OrderRepository`, never an `OrderItemRepository`.
You load and save whole orders, never a single item.

**3. Point at other aggregates by ID.**
A `User` is the root of its own aggregate, so an Order holds only the ID.

```go
type Order struct {
    userID int64 // not a *User
}
```

Holding the whole object tempts you to change a User while saving an Order.
That would be two aggregates changing in one step.

---
## Repository

==Repository== gives collection-like access to aggregates.
It hides the storage details from the domain.
It is where DDD meets ports and adapters: the repository is a port.

---
## Related Notes

- [[11-domain-driven-design]] — the methodology these serve
- [[5-ports-and-adapters]] — the Repository as a port
- [[4-domain-vs-infrastructure]] — these objects live in the domain

## Review History

- 2026-08-05
