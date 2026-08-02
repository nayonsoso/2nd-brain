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
## Package Structure

The port and the adapter must live in different places.
==The port lives inside the core. The adapter lives outside the core.==

```
core/
  domain/       # entities
  port/         # PaymentPort  <- the interface
  usecase/
infrastructure/
  payment/      # TossPaymentAdapter  <- the implementation
  persistence/  # MySqlOrderAdapter
```

This split makes the dependency point in one direction only.
==Infrastructure imports core. Core never imports infrastructure.==
If you ever see an import from `core` to `infrastructure`, the pattern is broken.

---
## Inbound vs Outbound Ports

==Inbound (driving)== ports are how the outside calls the core.
For example, an HTTP controller calls a use case through an inbound port.
The outside world drives the core.

==Outbound (driven)== ports are how the core calls the outside.
For example, the core saves data through a repository port.
The core does not know if it is MySQL or Postgres behind the port.

![[Pasted image 20260802094115.png|1000]]

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
## Java Example

The same idea in Java is an ==interface (port)== and a ==class that implements it (adapter)==.

```java
// Port — lives in the core package
public interface PaymentPort {
    void pay(int amount);
}

// Adapter — lives in the infrastructure package
public class TossPaymentAdapter implements PaymentPort {
    private final RestClient client;

    @Override
    public void pay(int amount) {
        // call the Toss payment API
    }
}
```

In Spring, the adapter is a `@Component` and the core gets it by constructor injection.
The core field is typed `PaymentPort`, never `TossPaymentAdapter`.
See [[3-dependency-injection]] for how the adapter is handed to the core.

---
## Real Java Libraries

Java already uses this pattern in tools you use every day.

| Port (interface) | Adapter (implementation) |
| ---------------- | ------------------------ |
| `java.sql.Driver`, `java.sql.Connection` | MySQL Connector/J, PostgreSQL JDBC Driver |
| `slf4j-api` | Logback, Log4j2 |

Your code only talks to `Connection`.
It does not know if MySQL or Postgres is behind it.
To change the database, you swap the driver, not your code.
SLF4J is even clearer: the bridge libraries are literally named `*-adapter`.

---
## Not Every Interface Is a Port

This is the part people get wrong.
`List` and `ArrayList` are just polymorphism, not a port.
==A port must be defined by the core, in the core's own language.==

```java
// Port — talks in domain words
void save(Order order);

// Not a port — DB words leaked into the core
void executeQuery(String sql);
```

The second one forces the core to know about SQL.
When the outside world's words appear in the interface, the port is already broken.

---
## Related Notes

- [[3-dependency-injection]] — how the adapter is supplied to the core
- [[6-use-case]] — what sits behind an inbound port
- [[7-dto-and-mappers]] — how data crossing a port is shaped
- [[9-hexagonal-architecture]] — the pattern built on ports and adapters

## Review History
- 2026-08-02