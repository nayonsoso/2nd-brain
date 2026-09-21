#understand #ai-draft

# Detaching a Context

## The Bug This Solves

You write an endpoint that starts slow work in the background.

```go
func handler(w http.ResponseWriter, r *http.Request) {
    ctx := r.Context()

    go recalculateAllPrices(ctx)   // takes 3 minutes

    w.WriteHeader(http.StatusAccepted)
}
```

You deploy it. The logs fill with `context canceled`.
The job never finishes. Sometimes it writes half the rows and stops.

==Nothing crashed. The context did exactly what it was designed to do.==

---
## Why It Happens

From [[5-go-context]]: `net/http` cancels `r.Context()` ==when the handler returns==.

So the order is:

1. The handler starts the goroutine.
2. The handler writes `202` and returns.
3. `net/http` cancels `r.Context()`.
4. The goroutine's next DB call sees a dead context and fails.

The handler returned in 2 milliseconds.
The background job got 2 milliseconds to live.

==Cancellation flows down the tree, and your goroutine is a child.==

---
## The Wrong Fix

```go
go recalculateAllPrices(context.Background())
```

This stops the cancellation. It also throws away everything else.

Gone:
- the trace ID, so this work no longer appears in your traces
- the request-scoped logger and its fields
- the user or tenant stored in the context
- any locale or feature-flag value a middleware put there

==You fixed the lifetime and broke the observability.==
Now a failing background job is a log line with no request to tie it to.

---
## The Right Fix: Detach

Detaching means:

> ==Keep the values. Drop the cancellation and the deadline.==

The result is a context that still knows *who* and *why*, but no longer dies when the HTTP response is sent.

| | Values kept? | Dies with the request? |
|--|--|--|
| `r.Context()` | yes | ==yes== — the bug |
| `context.Background()` | ==no== — the other bug | no |
| Detached context | yes | no |

---
## The Standard Library Way

Go 1.21 added this to the standard library.

```go
ctx := context.WithoutCancel(r.Context())
go recalculateAllPrices(ctx)
```

`context.WithoutCancel(parent)` returns a context that:
- returns the same values as the parent,
- has no deadline,
- and is ==never cancelled==.

==If you are on Go 1.21 or newer, this is the default answer.==

---
## The Library Way: `onecontext.Detach`

Some code uses `github.com/teivah/onecontext`, which predates the standard library function.

```go
taskCtx, cancel := onecontext.Detach(ctx)
```

It does the same thing, plus one extra.

| | `context.WithoutCancel` | `onecontext.Detach` |
|--|--|--|
| Keeps values | yes | yes |
| Has a deadline | no | no |
| Cancelled by the parent | no | no |
| Gives you a `cancel` | ==no== | ==yes== |

The difference is that second return value.
`WithoutCancel` gives a context that can *never* be cancelled.
`Detach` gives you your own cancel function, so ==you== can stop the work later — on shutdown, or when a user asks to abort the task.

If you ignore the cancel with `_`, the two are effectively the same:

```go
taskCtx, _ := onecontext.Detach(ctx)
```

---
## The Cost Of Detaching

You removed the safety net. Say it out loud:

==Nothing will ever stop this work automatically.==

- No request timeout.
- No cancel when the user disconnects.
- No cancel when the parent gives up.

So you must add your own limit. A detached context with a fresh timeout is the usual shape:

```go
taskCtx := context.WithoutCancel(r.Context())
taskCtx, cancel := context.WithTimeout(taskCtx, 10*time.Minute)

go func() {
    defer cancel()
    recalculateAllPrices(taskCtx)
}()
```

Read the order carefully.
==Detach first, then add your own timeout.==
The new timeout is yours. It has nothing to do with the request's timeout.

---
## When To Detach

| Situation | Detach? |
|-----------|---------|
| Background job that must finish after the response | ==Yes== |
| Writing an audit log or metric after replying | Yes |
| Cleanup that must run even if the client left | Yes |
| A normal DB read inside the handler | ==No== |
| Calling another service to build the response | No |

The test is simple.
==If the user leaving should stop the work, do not detach.==

---
## Related Notes

- [[5-go-context]] — what you are detaching from
- [[4-fire-and-forget]] — the pattern this makes possible
- [[7-background-task-pattern]] — the whole thing assembled

## Review History

-
