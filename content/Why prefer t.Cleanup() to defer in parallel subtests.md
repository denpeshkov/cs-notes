---
tags:
  - Go
---

# Overview

Consider the following example where subtests using `t.Run` make `t.Parallel` calls to run in parallel:

```go
func TestFoo(t *testing.T) {
	t.Log("TestFoo entered")
	defer t.Log("TestFoo exited")

	t.Run("Sub1", func(t *testing.T) {
		t.Parallel()

		t.Log("Sub1 entered")
		defer t.Log("Sub1 exited")
	})
	t.Run("Sub2", func(t *testing.T) {
		t.Parallel()

		t.Log("Sub2 entered")
		defer t.Log("Sub2 exited")
	})
}
```

The output is as follows:

```
=== RUN   TestFoo
    a_test.go:6: TestFoo entered
=== RUN   TestFoo/Sub1
=== PAUSE TestFoo/Sub1
=== RUN   TestFoo/Sub2
=== PAUSE TestFoo/Sub2
=== NAME  TestFoo
    a_test.go:21: TestFoo exited
=== CONT  TestFoo/Sub1
=== CONT  TestFoo/Sub2
    a_test.go:18: Sub2 entered
    a_test.go:20: Sub2 exited
=== NAME  TestFoo/Sub1
    a_test.go:12: Sub1 entered
    a_test.go:14: Sub1 exited
--- PASS: TestFoo (0.00s)
    --- PASS: TestFoo/Sub2 (0.00s)
    --- PASS: TestFoo/Sub1 (0.00s)
PASS
```

As seen, the deferred log in `TestFoo` executes **before** either `Sub1` or `Sub2` finish running

The issue with Go's `t.Parallel()` is that it requires an initial pass to determine which tests can run in parallel, pausing the test goroutines and continuing them later. When used in subtests, the top-level test may finish and execute its `defer`statements while subtests are still paused, which is exactly what happens in our case

The solution is to use `t.Cleanup`, which, unlike `defer`, ensures the correct execution order even with `t.Parallel()`. It guarantees that cleanup tasks are run at the right time, after the parallel subtests have finished

# References

- [Why to prefer \`t.Cleanup\` to \`defer\` in tests with subtests using \`t.Parallel\` — brandur.org](https://brandur.org/fragments/go-prefer-t-cleanup-with-parallel-subtests)
- [testing package - testing - Go Packages](https://pkg.go.dev/testing#T.Cleanup)
- [The Go Programming Language Specification - The Go Programming Language](https://go.dev/ref/spec#Defer_statements)
