#understand #ai-draft

# Use Case

A ==use case== is an object that runs one application action.
It lives **just behind an inbound port.**
Think of it as the thing a controller calls to get work done.

---
## What It Does

A use case ==orchestrates== the core business logic for one action.
For example: "create an order" or "cancel a booking".
It does not hold business rules itself.
It **calls domain objects and outbound ports** (like a repository) to get the job done.

---
## Where It Sits

The flow looks like this: controller → inbound port → use case → domain objects / outbound ports.
A controller (HTTP, CLI, etc.) is an adapter.
It calls a use case through an inbound port.
The use case does not know about HTTP, JSON, or any outside detail.
It only knows about the domain and the ports it depends on.

---
## Go Example

```go
// Use case: orchestrates one application action
type CreateOrderUseCase struct {
    orders OrderRepository // outbound port
}

func (uc CreateOrderUseCase) Execute(sku string, count int) error {
    order := Order{SKU: sku, Count: count} // domain object
    return uc.orders.Save(order)           // calls outbound port
}
```

The `CreateOrderUseCase` does not know if `OrderRepository` is backed by MySQL or Postgres.
It only knows the port.
A controller calls `Execute` and does not know how the order is built or saved.

---
## Other Names

Some patterns use different words for this same idea.
Clean Architecture calls it an ==Interactor==.
Some codebases call it an ==Application Service==.
The name differs, but the job is the same: run one action, orchestrate the core.

---
## Related Notes

- [[5-ports-and-adapters]] — the inbound port a use case sits behind
- [[4-domain-vs-infrastructure]] — the domain objects a use case orchestrates
- [[10-clean-architecture]] — where "Use Cases" is a named layer

## Review History

- 2026-08-02
