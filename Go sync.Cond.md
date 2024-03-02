---
tags: Go Concurrency
---

# Wait in a Loop

We should use `Cond.Wait()` in a loop like so:

```go
c.L.Lock()
for !condition() {
    c.Wait()
}
// make use of condition
c.L.Unlock()
```

From the documentation:

> `Wait` atomically unlocks `c.L` and suspends execution of the calling goroutine. After later resuming execution, `Wait` locks `c.L` before returning.

The unlocking and suspending are **atomic** on entrance, but wake-up (resuming) and locking are **not atomic** on exit

The code for the `Wait`:

```go
func (c *Cond) Wait() {
	c.checker.check()
	t := runtime_notifyListAdd(&c.notify)
	c.L.Unlock()
	runtime_notifyListWait(&c.notify, t)
	c.L.Lock()
}
```

As we can see, `Wait` unlocks `c.L`, and then waits for notification and then locks `c.L` again. So the condition may change in some other goroutine between `c.L.Unlock()` and `c.L.Lock()`, in particular **after** `runtime_notifyListWait()` but **before** `c.L.Lock()`

Thus, we must check the condition in the loop to be able to wait again. Because after `Wait()` returns, we own the lock `c.L` and can be sure that the condition check in a loop is not racy

# Signal Scheduling

`sync.Cond` does not affect goroutine scheduling priority, and `Signal()` only makes the waiting goroutine available to be scheduled, but does not force it

For example, suppose we have one goroutine waiting on `Wait()` and other goroutines blocking on `Lock()` on the same `c.L`. Another goroutine currently holding the `c.L` calls `Signal()` and then `Unlock()` on `c.L`

Then after a `Signal()` (and `Unlock()`), every other waiting goroutine then contends (fairly) for the lock. Meaning, we cannot assume that the goroutine waiting on `Wait()` will be the first to execute

# References

- [sync package - sync - Go Packages](https://pkg.go.dev/sync#Cond)
- [concurrency - sync.Cond with Wait method in Go - Stack Overflow](https://stackoverflow.com/questions/76742838/sync-cond-with-wait-method-in-go)
- [sync package - sync - Go Packages](https://pkg.go.dev/sync#Cond)
- [Go advanced concurrency patterns: part 3 (channels) - Blog Title](https://blogtitle.github.io/go-advanced-concurrency-patterns-part-3-channels/)
- [Re: [go-nuts] Help with sync.Cond failing to signal](https://www.mail-archive.com/golang-nuts@googlegroups.com/msg47474.html)
