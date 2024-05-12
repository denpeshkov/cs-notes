---
tags: Go Concurrency OS_Arch
---

# Mutex Implementation

Go `sync.Mutex` is not a reentrant mutex #todo: [Reentrant mutex](https://groups.google.com/g/golang-nuts/c/XqW1qcuZgKg/m/Ui3nQkeLV80J)

## Basic Implementation

Early versions of Go used a simple [[Mutex|mutex]] implementation:

```go
// AddInt32 atomically adds delta to *val and returns the new value.
func AddInt32(val *int32, delta int32) (new int32) {...}

// Semacquire waits until *s > 0 and then atomically decrements it.
func Semacquire(s *uint32) {...}

// Semrelease atomically increments *s and notifies a waiting goroutine if one is blocked in Semacquire. 
func Semrelease(s *uint32) {...}

// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
type Mutex struct {
	// A user-level key.
	key  int32
	// A semaphore.
	sema uint32
}

// Lock locks m.
// If the lock is already in use, the calling goroutine blocks until the mutex is available.
func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.AddInt32(&m.key, 1) == 1 {
		// Changed from 0 to 1; we hold lock.
		return
	}
	// Mutex is already locked; wait on semaphore
	runtime.Semacquire(&m.sema)
}

// Unlock unlocks m.
// It is a run-time error if m is not locked on entry to Unlock.
//
// A locked Mutex is not associated with a particular goroutine.
// It is allowed for one goroutine to lock a Mutex and then
// arrange for another goroutine to unlock it.
func (m *Mutex) Unlock() {
	switch v := atomic.AddInt32(&m.key, -1); {
	case v == 0:
		// Changed from 1 to 0; no contention.
		return
	case v == -1:
		// Changed from 0 to -1: wasn't locked (or there are 4 billion goroutines waiting).
		panic("sync: unlock of unlocked mutex")
	}
	runtime.Semrelease(&m.sema)
}
```

Given implementation assumes support for [Fetch-and-Add](Atomic%20Instructions.md#Fetch-and-Add) and [semaphore](Semaphore.md) operations

The imple­mentation represents a mutex as two values: a user-level key `key` and a semaphore `sema`. The `key` counts the number of processes interested in holding the lock, including the one that does hold it:

 - If `key` is 0, the mutex is unlocked
 - If `key` is 1, the mutex is locked
 - If `key` is larger than 1, the mutex is locked by one process and there are `key`-1 processes waiting to acquire it. Those processes wait on the semaphore `sema`

The fast path (uncontended case) allows acquiring the mutex entirely in user space, without the need to acquire a [semaphore](Semaphore.md), thus avoiding a [system call](System%20Calls.md) and a possible [context switch](Context%20Switch.md)

This implementation enforces the FIFO policy due to the internal implementation of `runtime.Semacquire` and `runtime.Semrelease` based on a queue. If multiple goroutines are blocked on `sema` trying to acquire the mutex (`key` is greater than 1), then after calling `Unlock()`, the first goroutine blocked on `sema` will acquire the mutex. This means that a newcomer goroutine will acquire the lock only after all the blocked goroutines

## Allow Successive Acquisition

This implementation allows a goroutine to do successive acquisitions of a mutex even if there are blocked goroutines. Moreover, it allows a newcomer goroutine to acquire a mutex ahead of blocked goroutines (that is, it does not enforce FIFO)

On implementation level it's achieved by separating waiter count and locked flag

```go
// CompareAndSwapInt32 executes the compare-and-swap operation for an int32 value.
func CompareAndSwapInt32(addr *int32, old, new int32) (swapped bool) {...}

// Semacquire waits until *s > 0 and then atomically decrements it.
func Semacquire(s *uint32) {...}

// Semrelease atomically increments *s and notifies a waiting goroutine if one is blocked in Semacquire. 
func Semrelease(s *uint32) {...}

const (
	// Mutex is locked.
	mutexLocked = 1 << iota // 1
	// Goroutine woken to acquire mutex.
	mutexWoken // 2
	// Waiter count multiplier.
	mutexWaiterShift = iota // 2
)

// A Mutex is a mutual exclusion lock.
// The zero value for a Mutex is an unlocked mutex.
type Mutex struct {
	// A state of the mutex.
	state int32
	// A semaphore.
	sema  uint32
}

func (m *Mutex) Lock() {
	// Fast path: grab unlocked mutex.
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		// Changed from 0 to mutexLocked; we hold lock.
		return
	}
	
	// Slow path: try to acquire the mutex if it's not locked, otherwise increment waiters count, block and retry.
	awoke := false
	for {
		old := m.state
		// Set locked and increment waiters count if it's already locked.
		new := old | mutexLocked
		if old&mutexLocked != 0 {
			new = old + 1<<mutexWaiterShift
		}
		
		// The goroutine has been woken from sleep, so we need to reset the flag in either case.
		if awoke {
			new &^= mutexWoken
		}
		
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			// Changed from unlocked to locked; we hold lock.
			if old&mutexLocked == 0 {
				break
			}
			
			// Mutex is already locked; wait on semaphore.
			runtime.Semacquire(&m.sema)
			
			// We were woken up.
			awoke = true
		}
	}
}

func (m *Mutex) Unlock() {
	// Fast path: drop lock bit.
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if (new+mutexLocked)&mutexLocked == 0 {
		panic("sync: unlock of unlocked mutex")
	}

	old := new
	for {
		// If there are no waiters or a goroutine has already
		// been woken or grabbed the lock, no need to wake anyone.
		if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken) != 0 {
			return
		}
		
		// Grab the right to wake someone; decrement waiters count and mark as woken up.
		new = (old - 1<<mutexWaiterShift) | mutexWoken
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			runtime.Semrelease(&m.sema)
			return
		}
		old = m.state
	}
}
```

Given implementation assumes support for [Compare-and-Swap (CAS)](Atomic%20Instructions.md#Compare-and-Swap%20(CAS)), [Fetch-and-Add](Atomic%20Instructions.md#Fetch-and-Add) and [semaphore](Semaphore.md) operations

This implementation uses a `state` field instead of a `key` to separately track whether a mutex is locked (0th bit) and by how many processes (2nd bit and up). The 1st bit is used to track whether a goroutine is woken up

## Add Active Spinning #todo

Previous implementations are fully cooperative. That is, once contention is discovered, the goroutine calls into scheduler. This is suboptimal as the resource can become free soon after (especially if critical sections are short). Server software usually runs at ~50% CPU utilization, that is, switching to other goroutines is not necessary profitable.

This change adds limited active spinning if:

1. Running on a multicore machine and
2. `GOMAXPROCS` > 1 and
3. There is at least one other running [Processor](Go%20Goroutines%20and%20Scheduler%20Internals.md) `P` and
4. Local [run queue](Go%20Goroutines%20and%20Scheduler%20Internals.md) is empty

As opposed to runtime mutex don't do passive spinning, because there can be work on [global run queue](Go%20Goroutines%20and%20Scheduler%20Internals.md) on on other `P`s

```go



```

# RWMutex Implementation

# References

- [Source code: sync.Mutex basic implementation](https://github.com/golang/go/blob/12b7875bf2c534c7ec1659a733ec2d82a3f85076/src/pkg/sync/mutex.go)
- [Semaphores in Plan 9](http://swtch.com/semaphore.pdf)
- [Source code: sync.Mutex allow successive acquisition](https://github.com/golang/go/blob/dd2074c82acda9b50896bf29569ba290a0d13b03/src/pkg/sync/mutex.go)
- [Source code: sync.Mutex add active spinning](https://github.com/golang/go/blob/edcad8639a902741dc49f77d000ed62b0cc6956f/src/sync/mutex.go)
