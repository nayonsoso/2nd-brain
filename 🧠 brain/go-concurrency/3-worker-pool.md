#understand #ai-draft

# Worker Pool

## The Problem First

Goroutines are cheap, so people start one per job.

```go
for _, item := range items {
    go process(item)   // 100,000 items = 100,000 goroutines
}
```

This looks fine. It is not.
==Goroutines are cheap, but what they touch is not.==

- Each one may open a database connection. Your pool has maybe 20.
- Each one may call an external API. You get rate limited or banned.
- Each one holds memory until it finishes.
- You have no idea how many are running right now.

The work is unbounded.
==A worker pool puts a limit on it.==

---
## What Is a Worker Pool?

A worker pool is ==a fixed number of goroutines that read jobs from one shared queue==.

Three parts:

| Part | What it is |
|------|------------|
| Queue | A channel holding jobs waiting to run |
| Workers | N goroutines, started once, that loop forever |
| Job | A closure the worker calls |

The number of workers never changes.
The number of jobs can be huge.
==Concurrency stays capped at N, no matter how much work arrives.==

---
## The Whole Thing In 20 Lines

```go
type Task func()

type Pool struct {
    jobs chan Task
}

func NewPool(workers, queueSize int) *Pool {
    p := &Pool{
        jobs: make(chan Task, queueSize),
    }

    for i := 0; i < workers; i++ {
        go func() {
            for task := range p.jobs {  // blocks until a job arrives
                task()                  // run it
            }
        }()
    }

    return p
}

func (p *Pool) Submit(t Task) {
    p.jobs <- t
}
```

That is a real worker pool.
Read it again with [[1-goroutines-and-channels]] in mind.

- `make(chan Task, queueSize)` is the queue.
- The `for` loop starts N goroutines.
- `for task := range p.jobs` makes each worker wait for work.
- `Submit` drops a [[2-closures|closure]] into the queue.

Using it:

```go
pool := NewPool(10, 100)   // 10 workers, queue holds 100

pool.Submit(func() {
    calculatePrice(productID)
})
```

---
## How Workers Share The Queue

All workers read from the *same* channel.
Go guarantees ==each value goes to exactly one receiver==.

So you never need a lock here.
There is no "which worker gets this job" logic to write.
Whichever worker is free grabs the next job.

This is self-balancing.
A worker stuck on a slow job simply does not take new jobs.
The others keep going.

---
## What Happens When The Queue Is Full

This is the part people forget.

```go
pool := NewPool(10, 100)
// 100 jobs waiting, 10 running
pool.Submit(task)  // what now?
```

The channel send blocks.
==`Submit` will wait until a slot opens.==
This is called ==backpressure==. The system slows the caller down instead of falling over.

Sometimes blocking is wrong. In an HTTP handler it means a slow request.
Then you drop the job instead, using `select`:

```go
func (p *Pool) TrySubmit(t Task) error {
    select {
    case p.jobs <- t:
        return nil
    default:
        return errors.New("queue full")   // reject fast
    }
}
```

==You must choose: wait, or reject.== There is no third option.
A queue with no limit is not a third option. It just moves the crash to your memory.

---
## Getting Results Back

The pool above returns nothing.
If you need a result, the closure must carry a way to send it out.

```go
results := make(chan int, len(items))

for _, item := range items {
    item := item
    pool.Submit(func() {
        results <- process(item)
    })
}
```

The closure captures `results`.
That is exactly the closure behaviour from [[2-closures]].

If you do *not* need the result, you have chosen [[4-fire-and-forget]].

---
## How To Pick The Number Of Workers

It depends on what the work does.

| Work type | Bottleneck | Rough rule |
|-----------|-----------|------------|
| CPU heavy (math, parsing) | CPU cores | `runtime.NumCPU()` |
| I/O heavy (DB, HTTP) | The other system | Match the DB pool or the API rate limit |

==Do not guess big numbers.==
For I/O work the right number is usually set by the slowest downstream thing, not by your machine.

---
## When You Do Not Need A Pool

Do not build a pool for everything.

- A few known jobs → use `sync.WaitGroup` and plain goroutines.
- Jobs that can fail and should stop each other → use `errgroup`.
- Work that must survive a restart → use a real queue (Kafka, SQS, a DB table).

==A worker pool lives in memory. If the process dies, the queue is gone.==

---
## Related Notes

- [[1-goroutines-and-channels]] — the parts a pool is built from
- [[2-closures]] — what a job actually is
- [[4-fire-and-forget]] — submitting without waiting for the result
- [[7-background-task-pattern]] — a pool inside a web service

## Review History

-
