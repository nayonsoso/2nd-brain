#understand #ai-draft

# Fire-and-Forget

## Is This A Real Term?

Yes. ==It is a standard industry term, not Go slang.==
You will meet it in messaging, networking, database, and API documentation.

It comes from ==guided missiles==.
An older missile needed the plane to keep pointing a radar at the target.
A "fire-and-forget" missile guides itself.
The pilot launches it and flies away immediately.

Software borrowed the idea.
==You start the work and do not wait to see how it ends.==

---
## The Definition

A call is fire-and-forget when the caller:
- sends the work,
- returns immediately,
- and ==never learns the result==.

No return value. No error. No confirmation that it finished.
The opposite is ==request-response==, where you wait for an answer.

---
## In Go It Is One Keyword

```go
go sendAnalytics(event)   // fire-and-forget
```

That is it.
The `go` keyword starts the work and moves on.

Compare:

```go
err := sendAnalytics(event)   // request-response: you wait, you get an error
go sendAnalytics(event)       // fire-and-forget: you wait for nothing, you learn nothing
```

---
## The Same Idea In Other Places

The word shows up far outside Go.

| System | Fire-and-forget version |
|--------|------------------------|
| Network | ==UDP== — send the packet, no acknowledgement |
| Kafka | producer with `acks=0` — do not wait for the broker |
| RabbitMQ | publish without publisher confirms |
| HTTP | reply ==`202 Accepted`== instead of `200 OK` |
| Logging | write to a buffer, flush later |
| gRPC | a one-way call pattern |

==If you know the term, you can read all of these docs faster.==

---
## What You Trade Away

Fire-and-forget is fast because it skips guarantees.
Be honest about which ones you gave up.

| You lose | What it means |
|----------|---------------|
| The return value | You cannot use the result |
| The error | A failure is silent unless you log it |
| Completion | You do not know if it finished |
| Ordering | Two jobs may finish in any order |
| Durability | A restart kills anything still running |

==Silence is the real danger.==
A synchronous call that fails gives you a stack trace.
A fire-and-forget call that fails gives you nothing, unless you wrote the logging yourself.

---
## When It Is The Right Choice

Good fits:
- The caller genuinely does not need the answer.
- Losing one item is acceptable (analytics, metrics, a nice-to-have email).
- The work is slow and the user should not wait for it.

Bad fits:
- Money. Payments. Stock counts.
- Anything the user will ask about later.
- Anything that must not be lost.

==Ask one question: if this silently never runs, who notices and when?==
If the answer is "nobody, ever", it is not fire-and-forget. It is a bug you have not found yet.

---
## Making It Safer

Real systems rarely use pure fire-and-forget.
They keep a ==record== of the work, so it is not truly forgotten.

The usual shape:

1. Write a row to a `tasks` table with status `pending`.
2. Start the background work.
3. Return the ==task ID== to the caller right away.
4. The worker updates the row to `running`, then `done` or `failed`.
5. The caller polls `GET /tasks/{id}` to check.

```
POST /prices/recalculate   →   202 Accepted
                               { "process_id": "abc-123" }

GET  /processes/abc-123    →   200 OK
                               { "status": "running" }
```

==This is fire-and-forget for the HTTP connection, but not for the work.==
The connection is released in milliseconds.
The task still has a name, a status, and a history.

That is the pattern in [[7-background-task-pattern]].

---
## Related Notes

- [[1-goroutines-and-channels]] — `go` is the fire-and-forget keyword
- [[3-worker-pool]] — where the forgotten work actually runs
- [[6-detaching-context]] — why the work dies early if you skip this
- [[7-background-task-pattern]] — the safe version

## Review History

-
