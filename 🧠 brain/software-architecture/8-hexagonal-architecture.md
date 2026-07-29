#understand #ai-draft

# Hexagonal Architecture

## The Idea

==Hexagonal Architecture== splits a system into ==Inside (core)== and ==Outside (adapters)==.
The business core sits in the center.
All technology is pushed out to adapters that plug into ==ports==.
This pattern is also called "Ports and Adapters" architecture.

---
## Diagram

```
        [ HTTP Adapter ]        [ CLI Adapter ]
               \                     /
                \   (ports)          /
              [    BUSINESS CORE    ]
                /                   \
        [ Kafka Adapter ]      [ PG Adapter ]
```

---
## How It Works

The core defines ==ports== (interfaces).
Adapters implement those ports.
The core depends only on its own ports, never on a real database or framework.
This uses ==DIP== (Dependency Inversion Principle) to invert the dependency.

---
## Go Sketch

```go
// core depends on this port, not on a real DB
type OrderRepository interface {
    Save(o Order) error
}

// infrastructure adapter implements the port
type PostgresOrderRepository struct{ /* db handle */ }

func (r PostgresOrderRepository) Save(o Order) error { return nil }
```

The core defines the `OrderRepository` port.
The `PostgresOrderRepository` adapter lives outside the core and implements it.

---
## Related Notes

- [[2-solid-principles]] — DIP, the mechanism behind it
- [[5-ports-and-adapters]] — the building blocks
- [[4-domain-vs-infrastructure]] — what goes inside vs outside
- [[9-clean-architecture]] — the same idea, generalized

## Review History

-
