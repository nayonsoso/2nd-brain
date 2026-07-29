#understand #ai-draft

# Types of APIs

There are many kinds of APIs.
Each one works differently and is good for different use cases.

---
## REST

- The most common type.
- Uses ==HTTP methods== (GET, POST, PUT, DELETE).
- Data is usually in **JSON** format.
- Simple and easy to use.
- Good for most web and mobile apps.

→ See [[3-rest-api]] for more detail.

---
## GraphQL

- Made by Facebook.
- The client asks for ==exactly the data it needs==.
- One endpoint for everything (usually `/graphql`).
- Good when the client needs flexible or complex data.
- Harder to set up than REST.

---
## gRPC

- Made by Google.
- Uses ==Protocol Buffers== (binary format) instead of JSON.
- Very fast. Good for low-latency communication.
- Good for service-to-service calls inside a system (microservices).
- Not easy to use directly from a browser.

---
## WebSocket

- ==Keeps a connection open== between client and server.
- Both sides can send messages anytime.
- Good for real-time features like chat, live updates, or games.
- Different from REST — REST is request/response only.

---
## SOAP

- Old protocol. Uses ==XML==.
- Very strict rules and structure.
- Still used in banking, healthcare, and enterprise systems.
- Harder to work with than REST.

---
## Summary Table

| Type      | Format | Use Case                         |
| --------- | ------ | -------------------------------- |
| REST      | JSON   | General web/mobile APIs          |
| GraphQL   | JSON   | Flexible data fetching           |
| gRPC      | Binary | Fast microservice communication  |
| WebSocket | Any    | Real-time, two-way communication |
| SOAP      | XML    | Enterprise / legacy systems      |

---
## Related Notes

- [[1-what-is-api]] — what an API is
- [[3-rest-api]] — REST in detail
- [[14-rest-api-best-practices]] — how to design a good API

## Review History

- 2026-07-22
