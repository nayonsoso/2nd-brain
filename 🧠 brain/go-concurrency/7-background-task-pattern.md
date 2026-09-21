#understand #ai-draft

# The Background Task Pattern

## What This Note Is

Notes 1 to 6 taught six separate ideas.
This note puts them into ==one standard pattern== that Go web services use again and again.

The goal: ==an API that starts slow work and answers in milliseconds==.

---
## The Problem

A user asks you to recalculate prices for a whole country.
It takes 4 minutes.

You cannot hold the HTTP connection for 4 minutes.
- Load balancers cut idle connections, often after 60 seconds.
- The browser times out.
- A retry starts the same heavy job again.
- One slow request blocks a server slot.

==So the request and the work must be separated.==

---
## The Five Steps

| Step | What happens | Which note |
|------|--------------|-----------|
| 1 | Create a task record in the database, status `pending` | — |
| 2 | Wrap the real work in a closure | [[2-closures]] |
| 3 | Detach the request's context | [[6-detaching-context]] |
| 4 | Hand the closure to a worker pool | [[3-worker-pool]] |
| 5 | Return the task ID immediately | [[4-fire-and-forget]] |

---
## In Code

```go
func (s *Service) StartRecalculation(
    ctx context.Context,
    countryID string,
) (TaskID, error) {

    // 1. record it, so the work is never truly "forgotten"
    task := Task{ID: uuid.New(), Status: "pending"}
    if err := s.repo.Save(ctx, task); err != nil {
        return "", err          // failed before starting: a normal error
    }

    // 3. keep the trace ID, drop the request's cancellation
    taskCtx := context.WithoutCancel(ctx)

    // 2 + 4. package the work, hand it to the pool
    s.pool.Submit(func() {
        err := s.recalculate(taskCtx, countryID)
        s.repo.MarkFinished(taskCtx, task.ID, err)
    })

    // 5. return now, not in 4 minutes
    return task.ID, nil
}
```

==Everything before `Submit` is synchronous.== If saving the record fails, the caller gets a real error.
==Everything inside the closure is asynchronous.== Its failures live in the database row, not in the HTTP response.

---
## The HTTP Side

```
POST /prices/recalculate
  → 202 Accepted
    { "process_id": "8f14e45f" }

GET /processes/8f14e45f
  → 200 OK
    { "status": "running" }
```

==`202 Accepted` is the correct status code.==
It means "I took your request, I have not done it yet".
`200 OK` would be a lie, because nothing is done.

---
## Why Each Piece Is There

Remove any one piece and something breaks.

| Piece | Remove it and... |
|-------|-----------------|
| Task record | the caller can never find out what happened |
| Closure | the pool would need to know about your business logic |
| Detached context | ==the job dies the moment the response is sent== |
| Worker pool | 1,000 requests start 1,000 jobs and kill the database |
| Immediate return | the connection times out anyway |

The ==detached context== is the one that is easy to miss.
Everything looks correct in review, and it fails only under real traffic.

---
## The Order Matters

Detach ==before== submitting, not inside the worker.

```go
taskCtx := context.WithoutCancel(ctx)   // on the request goroutine
s.pool.Submit(func() {
    use(taskCtx)                        // already safe
})
```

If you pass the raw `ctx` into the closure, the race is already lost.
The handler may return before the worker even picks the job up.

---
## What This Pattern Does Not Give You

Be honest about the limits.

- ==The queue is in memory.== A pod restart loses every pending job.
- A job that was `running` when the process died stays `running` forever.
- Nothing retries automatically.

If losing jobs is unacceptable, you need a real queue (Kafka, SQS, or a polled database table).
==This pattern is for work that is slow and important, but survivable if it is lost once.==

A common repair: on startup, find rows stuck in `running` and mark them `failed`.

---
## A Real Example

In `pricing-tool`, `processTracker.NewTask(...)` is exactly this pattern.

```go
func (t *Tracker) NewTask(
    ctx context.Context,
    operationName string,
    marketID, parentID uuid.UUID,
    fn func(taskCtx context.Context) error,
) (*entity.ProcessTracker, error)
```

Reading it against the five steps:

1. It builds a `ProcessTracker` row with `IsSuccessful: false` and upserts it.
2. `fn` is the closure the caller passes in.
3. `taskCtx, _ := onecontext.Detach(ctx)` cuts the request's cancellation. The `_` throws away the cancel handle, so this behaves like `context.WithoutCancel`.
4. `AddWorkNonBlocking(task)` submits to the pool. ==Non-blocking== means it rejects instead of waiting when the queue is full — the `TrySubmit` choice from [[3-worker-pool]].
5. It returns `&process` right away, so the API can reply with the process ID.

==Same five steps, different names.==

---
## Related Notes

- [[1-goroutines-and-channels]] — the base
- [[2-closures]] — the job
- [[3-worker-pool]] — the runner
- [[4-fire-and-forget]] — the trade-off
- [[5-go-context]] — the signal
- [[6-detaching-context]] — the fix that makes this work

## Review History

-
