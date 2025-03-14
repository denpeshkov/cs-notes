---
tags:
  - Go
  - Concurrency
  - TODO
---

# Worker Pool

The classical worker pool in Go can be implemented as follows: 

```go
taskCh := make(chan Task, n) // Can also be unbuffered
var wg sync.WaitGroup

wg.Add(n)
for range n {
	go func() {
		defer wg.Done()
		for task := range taskCh {
			perform(task)
		}
	}()
}

for _, task := range tasks {
	taskCh <- task
}

close(taskCh)
wg.Wait()
```

This worker pool implementation has a problem with idle workers (goroutines) - until the `taskCh` is drained of work all the goroutines are still alive in the background and consume memory. Also, in case of a `panic` all those idling goroutines will pollute the goroutine stacktrace dump

We should start the goroutine only when we have a work to do, and exit as soon as the work is done:

```go
sem := make(chan struct{}, n)
for _, task := range tasks {
	sem <- struct{}
	go func(task Task) {
		perform(task)
		<-sem
	}(task)
}

for i := range n {
	sem <- struct{}
}
```

# Optimal Number of Goroutines

The optimal number of goroutines in the pool depends on the workload type:

- If the workload is I/O-bound, the value mainly depends on the external system
- If the workload is CPU-bound, the optimal number of goroutines is close to the number of available threads `GOMAXPROCS`

# References

- [GopherCon 2018: Rethinking Classical Concurrency Patterns - Bryan C. Mills - YouTube](https://youtu.be/5zXAHh5tJqQ?si=0YVSxsiCj71RACER)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
