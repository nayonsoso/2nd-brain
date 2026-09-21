#essential #ai-draft

# Goroutines and Channels

## Why This Note Comes First

A worker pool is not a special Go feature.
It is just ==goroutines and channels put together==.
So you need these two things first.

---
## What Is a Goroutine?

A ==goroutine is a function that runs at the same time as the rest of your code==.
You start one with the `go` keyword.

```go
go sendEmail(user)   // starts now, does not block
fmt.Println("done")  // runs immediately, does not wait
```

The `go` line returns right away.
`sendEmail` keeps running in the background.

---
## Goroutines Are Cheap

An OS thread costs about 1 MB of memory.
A goroutine starts at about ==2 KB==.
Go runs many goroutines on a few OS threads.
The Go runtime decides which goroutine runs on which thread.

This is why Go programs can have thousands of goroutines.
In Java or C you would never start 10,000 threads.
In Go, 10,000 goroutines is normal.

---
## The Problem With `go`

`go` gives you no way to get anything back.

```go
go calculatePrice(item)
// Where is the result? Gone.
// Did it fail? You do not know.
// Is it finished? You do not know.
```

You threw the work away.
You need another tool to talk to that goroutine.
That tool is a channel.

---
## What Is a Channel?

A ==channel is a pipe that carries values between goroutines==.
One goroutine puts a value in. Another goroutine takes it out.

```go
ch := make(chan int)  // a pipe that carries ints

go func() {
    ch <- 42          // send 42 into the pipe
}()

result := <-ch        // take a value out of the pipe
fmt.Println(result)   // 42
```

The arrow `<-` always points in the direction the data moves.

---
## Channels Block

This is the most important rule.

A send waits until someone receives.
A receive waits until someone sends.
==Blocking is how goroutines stay in sync==.

```go
ch := make(chan int)
ch <- 1   // deadlock! nobody is receiving
```

This program hangs forever.
The send has no partner.

---
## Buffered Channels

A buffered channel has room inside it.
Sends do not block until the buffer is full.

```go
ch := make(chan int, 3)  // room for 3 values

ch <- 1  // ok, no waiting
ch <- 2  // ok
ch <- 3  // ok, buffer now full
ch <- 4  // blocks! waits for someone to receive
```

==The buffer size is the size of your queue.==
This matters a lot for worker pools.

| Type | `make(chan T)` | `make(chan T, n)` |
|------|----------------|-------------------|
| Name | Unbuffered | Buffered |
| Send blocks | Always, until a receiver is ready | Only when the buffer is full |
| Use for | Handoff, strict sync | A job queue with a limit |

---
## Closing and Ranging

The sender closes a channel to say "no more values".
A `for range` over a channel reads until it is closed.

```go
ch := make(chan int, 3)
ch <- 1
ch <- 2
close(ch)

for v := range ch {   // reads 1, then 2, then the loop ends
    fmt.Println(v)
}
```

Rules to remember:
- ==Only the sender closes a channel.== Never the receiver.
- Sending on a closed channel panics.
- Receiving from a closed channel returns the zero value immediately.

---
## Putting Them Together

This is the whole idea in one small example.

```go
jobs := make(chan int, 5)
results := make(chan int, 5)

// one goroutine doing work
go func() {
    for n := range jobs {
        results <- n * 2
    }
    close(results)
}()

jobs <- 1
jobs <- 2
close(jobs)

for r := range results {
    fmt.Println(r)  // 2, then 4
}
```

==You just built a worker pool with one worker.==
A real pool is the same code with more goroutines.

---
## Related Notes

- [[2-closures]] — how you package work into a value you can send
- [[3-worker-pool]] — the same idea with many workers
- [[5-go-context]] — how you tell a goroutine to stop

## Review History

-
