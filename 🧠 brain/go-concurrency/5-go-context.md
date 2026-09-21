#essential #ai-draft

# Go Context

## What Is It?

`context.Context` is the value you see as the first argument almost everywhere in Go.

```go
func GetPrice(ctx context.Context, id string) (Price, error)
```

It does ==two jobs, and only two==:

1. ==Carry a stop signal== — cancellation and deadlines.
2. ==Carry request-scoped values== — trace ID, logger, user ID.

That is the whole type. Everything else is built on these two.

---
## Why Job 1 Exists

A request comes in. Your handler calls the database. The database is slow.
The user closes the browser.

Without a context, your code keeps working for nobody.
The query still runs. The connection is still held.
==Context is how "stop, nobody is waiting" travels down through every function call.==

---
## The Context Tree

Every context has a parent.
The root is `context.Background()`.

```
context.Background()
  └── r.Context()                    (created by net/http per request)
        └── context.WithTimeout(5s)  (your DB call)
              └── passed into the driver
```

==Cancellation flows down, never up.==
Cancel a parent and every child is cancelled.
Cancel a child and the parent is untouched.

This is the single most important rule.
Everything in [[6-detaching-context]] is about breaking this link on purpose.

---
## The Functions You Actually Use

```go
// Roots
ctx := context.Background()   // real root, use in main() and tests
ctx := context.TODO()         // "I will fix this later" marker

// Cancellation
ctx, cancel := context.WithCancel(parent)      // you cancel it by hand
ctx, cancel := context.WithTimeout(parent, 5*time.Second)
ctx, cancel := context.WithDeadline(parent, someTime)
defer cancel()                                 // ALWAYS

// Values
ctx := context.WithValue(parent, userKey, user)
user := ctx.Value(userKey)
```

==Always `defer cancel()`.==
Not calling it leaks the timer and the goroutine behind it.
`go vet` will warn you.

---
## How You Listen For The Signal

Two ways.

Check if it is already dead:

```go
if err := ctx.Err(); err != nil {
    return err   // context.Canceled or context.DeadlineExceeded
}
```

Wait for it while doing something else:

```go
select {
case result := <-resultCh:
    return result, nil
case <-ctx.Done():          // closed when cancelled or timed out
    return nil, ctx.Err()
}
```

`ctx.Done()` is just a channel.
==It is closed, not written to.== A closed channel makes every receiver wake up at once.
That is how one cancel stops a hundred goroutines.

---
## Context In An HTTP Server

This is the part that matters for background work.

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()
    ...
}
```

Go's `net/http` creates this context for you.
==It is cancelled when the handler returns, or when the client disconnects.==

That is correct and useful for normal work.
It becomes a problem the moment you start something that should outlive the response.
See [[6-detaching-context]].

---
## The Rules

These are community conventions, but they are followed everywhere.

| Rule | Why |
|------|-----|
| First parameter, always named `ctx` | Every Go reader expects it there |
| Never store a Context in a struct | It belongs to one request, not to an object |
| Never pass `nil` — use `context.TODO()` | `nil` panics later, far from the cause |
| Only request-scoped values | It is not a bag for dependencies |
| Pass it down to every blocking call | An ignored context cannot cancel anything |

The value rule needs care.
==A trace ID belongs in a context. A database handle does not.==
If the function cannot work without it, pass it as a normal argument.

---
## Values Are Untyped, So Use A Private Key

`ctx.Value` takes and returns `any`.
Two packages using the string `"user"` would collide.

```go
type ctxKey struct{}          // unexported, unique to this package

ctx = context.WithValue(ctx, ctxKey{}, user)
user, ok := ctx.Value(ctxKey{}).(User)
```

==Never use a plain string as a context key.==

---
## Related Notes

- [[1-goroutines-and-channels]] — `ctx.Done()` is a channel
- [[6-detaching-context]] — cutting the cancellation link
- [[7-background-task-pattern]] — where all of this is used together

## Review History

-
