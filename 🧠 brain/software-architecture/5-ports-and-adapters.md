#understand #ai-draft

# Ports and Adapters

==Ports and Adapters== is a way to protect the core business logic from the outside world.
The core defines what it needs. The outside world adapts to fit.

## Port

A ==Port== is a standardized interface defined by the core.
Think of it as a hole in the wall of the business logic.
The core says: "I need something that can do X."
It does not care how X is done.

---
## Adapter

An ==Adapter== is the concrete implementation that fits into a port.
Think of it as the plug that goes into the hole.
The adapter lives outside the core, in the infrastructure layer.
Examples: a MySQL adapter, an external payment gateway (PG) adapter.

---
## Inbound vs Outbound

==Inbound (driving)== ports are how the outside calls the core.
For example, an HTTP controller calls a use case through an inbound port.
The outside world drives the core.

==Outbound (driven)== ports are how the core calls the outside.
For example, the core saves data through a repository port.
The core does not know if it is MySQL or Postgres behind the port.

---
## Go Example

```go
// Port (defined by the core)
type PaymentPort interface {
    Pay(amount int) error
}

// Adapter (infrastructure implements the port)
type TossPaymentAdapter struct{ /* client, config */ }

func (a TossPaymentAdapter) Pay(amount int) error {
    // call the Toss payment API
    return nil
}
```

The core depends only on the `PaymentPort` interface, not on any specific service.
The `TossPaymentAdapter` lives in infrastructure and implements the port.
To switch payment providers, you only replace the adapter.

---
## Related Notes

- [[3-dependency-injection]] — how the adapter is supplied to the core
- [[6-use-case]] — what sits behind an inbound port
- [[7-dto-and-mappers]] — how data crossing a port is shaped
- [[9-hexagonal-architecture]] — the pattern built on ports and adapters

## Review History

-
