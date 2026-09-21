#essential #ai-draft

# Closures

## Simple Definition

A ==closure is a function that remembers the variables around it==.
It "closes over" those variables and carries them along.
When you call it later, those variables are still there.

In Go, functions are values.
You can store them, pass them, and return them.
A closure is a function value with memory attached.

---
## The Smallest Example

```go
func counter() func() int {
    count := 0            // a normal local variable

    return func() int {   // this inner function is a closure
        count++
        return count
    }
}

c := counter()
fmt.Println(c())  // 1
fmt.Println(c())  // 2
fmt.Println(c())  // 3
```

`count` is a local variable inside `counter`.
Normally it would disappear when `counter` returns.
But the inner function uses it, so ==Go keeps it alive==.
The variable lives as long as the closure lives.

---
## Capture Is By Reference, Not By Copy

This surprises people.
The closure shares the *same* variable, not a copy of it.

```go
x := 10
f := func() { fmt.Println(x) }

x = 20
f()  // prints 20, not 10
```

The closure did not save the value `10`.
It saved ==a link to the variable `x`==.

---
## Why Closures Matter For Background Work

A worker pool needs one thing: a job it can run later.
But a job needs data. Which user? Which price? Which order ID?

Without closures you would need a struct and an interface.

```go
type Job interface{ Run() }

type PriceJob struct {
    ProductID string
    Country   string
}

func (j PriceJob) Run() { calculate(j.ProductID, j.Country) }
```

With a closure you get the same thing in one line.

```go
job := func() {
    calculate(productID, country)
}
```

==The closure carries the data and the behaviour together.==
The worker pool does not need to know anything about prices.
It only needs to know how to call `func()`.

---
## The Common Job Type

This is why you see this type everywhere in Go.

```go
type Task func()              // no input, no output
type Task func() error        // can report a failure
type Task func(ctx context.Context) error   // can also be stopped
```

The pool accepts `Task`.
The caller builds the `Task` as a closure.
Neither side knows about the other's business logic.

```go
pool.Submit(func() error {
    return service.RecalculatePrices(productID)
})
```

---
## The Loop Variable Trap

This is the classic Go bug.
Every Go developer hits it once.

```go
for _, user := range users {
    go func() {
        sendEmail(user)   // which user?
    }()
}
```

Before ==Go 1.22==, all goroutines shared one `user` variable.
The loop finished fast. The goroutines started slow.
So most of them saw the *last* user.

The old fix was to pass it in as an argument.

```go
for _, user := range users {
    go func(u User) {
        sendEmail(u)      // u is a real copy, safe
    }(user)
}
```

Since ==Go 1.22 each loop iteration gets its own variable==, so the first version is now correct.
But you will still see the old fix in real code.
And the deeper lesson stays true: ==a closure shares variables, so be careful what you share==.

---
## Where You Already Use Closures

You have seen closures many times without naming them.

| Place | Example |
|-------|---------|
| `defer` | `defer func() { tx.Rollback() }()` |
| HTTP middleware | a handler that wraps another handler |
| Sorting | `sort.Slice(xs, func(i, j int) bool { ... })` |
| Table tests | `t.Run(name, func(t *testing.T) { ... })` |
| Background jobs | `pool.Submit(func() { ... })` |

---
## Related Notes

- [[1-goroutines-and-channels]] — what actually runs the closure
- [[3-worker-pool]] — where closures become jobs
- [[7-background-task-pattern]] — the full pattern

## Review History

-
