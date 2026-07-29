#understand #ai-draft

# DTO and Mappers

Data moves between layers of your system.
==DTOs== and ==mappers== control how that data crosses each boundary.

## DTO

==DTO (Data Transfer Object)== is a plain object used to move data across a boundary.
It carries data only.
It has no business logic.

---
## Mapper

==Mapper== is code that converts between DTOs and domain objects.
It reads a DTO and builds a domain object, or the other way around.

---
## Why We Need Them

Outside data (JSON from a request, rows from a DB) must not flow straight into the business logic.
The DTO and mapper act as a ==firewall== that keeps each layer independent.
If the outside shape changes, only the mapper changes.
The business logic stays untouched.

---
## Go Example

```go
type CreateOrderRequest struct { // DTO from the outside
    SKU   string `json:"sku"`
    Count int    `json:"count"`
}

func toOrder(r CreateOrderRequest) Order { // mapper
    return Order{SKU: r.SKU, Count: r.Count}
}
```

The struct is the DTO — it holds the raw input from outside.
The function is the mapper — it converts that input into a domain object.

---
## Related Notes

- [[5-ports-and-adapters]] — DTOs travel through ports
- [[4-domain-vs-infrastructure]] — the boundary DTOs protect

## Review History

-
