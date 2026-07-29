#essential #ai-draft

# Dependency Injection

## What is DI

==Dependency Injection (DI)== means giving an object its dependencies from outside.
You do not create them inside the object.
Something else builds the dependency and passes it in.

---
## Go Example

The `Service` does not build its own repository.
It receives one that already satisfies the interface.
This keeps `Service` decoupled from any specific implementation.

```go
type Repository interface {
    Save(o Order) error
}

type Service struct {
    repo Repository
}

func NewService(repo Repository) *Service { // dependency passed in
    return &Service{repo: repo}
}
```

---
## DIP vs DI

People often mix up DIP and DI.
==DIP is a design principle==: depend on abstractions, not concrete types.
==DI is a technique==: pass dependencies in from outside.
DI is the most common way to achieve DIP.

---
## Why It Helps Testing

Dependencies come from outside the object.
In a test, you pass a fake or mock that satisfies the interface.
The test does not need a real database or network.
This makes unit tests easy and fast.

---
## Related Notes

- [[2-solid-principles]] — DIP, the principle DI helps achieve
- [[5-ports-and-adapters]] — where injected interfaces become ports

## Review History

-
