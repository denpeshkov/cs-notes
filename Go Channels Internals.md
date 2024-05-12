---
tags:
  - Go
  - TODO
---

# Overview

At a high level a channel is a pointer to the struct with a buffer, two wait queues and a lock

## The `hchan` Struct

A channel is a **pointer** to a `hchan` struct allocated on the [heap](Heap%20Memory.md):

```go
type hchan struct {
	qcount   uint           // total data in the queue
	dataqsiz uint           // size of the circular queue
	buf      unsafe.Pointer // points to an array of dexataqsiz elements
	elemsize uint16
	closed   uint32
	timer    *timer // timer feeding this chan
	elemtype *_type // element type
	sendx    uint   // send index
	recvx    uint   // receive index
	recvq    waitq  // list of recv waiters
	sendq    waitq  // list of send waiters

	// lock protects all fields in hchan, as well as several
	// fields in sudogs blocked on this channel.
	//
	// Do not change another G's status while holding this lock
	// (in particular, do not ready a G), as this can deadlock
	// with stack shrinking.
	lock mutex
}

// waitq represents stores a taild and a head of the waiters list
type waitq struct {
	first *sudog
	last  *sudog
}
```

The main fields are:

- `buf` a circular FIFO queue storing elements of the channel (`sendx` and `recvx` are indexes to head and tail)
- `recvq` a [linked list](Linked%20List.md) of goroutines represented by [`sudog`](#The%20`sudog`%20Struct) struct waiting for receiving data from the channel
- `sendq` a [linked list](Linked%20List.md) of goroutines represented by [`sudog`](#The%20`sudog`%20Struct) struct waiting to send data on the channel
- `lock` a [runtime version of a mutex](Mutex.md) protecting fields in `hchan` and [`sudog`](#The%20`sudog`%20Struct)

Enqueueing an element to the `buf` causes a **copy** of the value to be added to the `buf`. Dequeuing an element from the `buf` causes a **copy** of the `buf` element to be assigned to the variable. It provides channels memory safety guarantees, as the only shared memory is a `hchan` protected by a `lock`

## The `sudog` Struct

Represents a goroutine (pseudo-goroutine) stored in a wait list:

```go
// sudog (pseudo-g) represents a g in a wait list, such as for sending/receiving
// on a channel.
//
// sudog is necessary because the g ↔ synchronization object relation
// is many-to-many. A g can be on many wait lists, so there may be
// many sudogs for one g; and many gs may be waiting on the same
// synchronization object, so there may be many sudogs for one object.
//
// sudogs are allocated from a special pool. Use acquireSudog and
// releaseSudog to allocate and free them.
type sudog struct {
	// The following fields are protected by the hchan.lock of the
	// channel this sudog is blocking on. shrinkstack depends on
	// this for sudogs involved in channel ops.

	g *g

	next *sudog
	prev *sudog
	elem unsafe.Pointer // data element (may point to stack)

	// The following fields are never accessed concurrently.
	// For channels, waitlink is only accessed by g.
	// For semaphores, all fields (including the ones above)
	// are only accessed when holding a semaRoot lock.

	acquiretime int64
	releasetime int64
	ticket      uint32

	// isSelect indicates g is participating in a select, so
	// g.selectDone must be CAS'd to win the wake-up race.
	isSelect bool

	// success indicates whether communication over channel c
	// succeeded. It is true if the goroutine was awoken because a
	// value was delivered over channel c, and false if awoken
	// because c was closed.
	success bool

	// waiters is a count of semaRoot waiting list other than head of list,
	// clamped to a uint16 to fit in unused space.
	// Only meaningful at the head of the list.
	// (If we wanted to be overly clever, we could store a high 16 bits
	// in the second entry in the list.)
	waiters uint16

	parent   *sudog // semaRoot binary tree
	waitlink *sudog // g.waiting list or semaRoot
	waittail *sudog // semaRoot
	c        *hchan // channel
}
```

The main fields are:

- `g` a pointer to the waiting [goroutine](Go%20Goroutines%20and%20Scheduler%20Internals.md)
- `elem` an element `g` is waiting to send/receive
- `next`/`prev` pointers to the next/previous `g` in a wait list `hchan.recvq`/`hchan.sendq`

# Channel Send

# Send on a Channel

Actions when [`G`](Go%20Goroutines%20and%20Scheduler%20Internals.md) sends on a non-full and non-empty channel:

1. Acquire the `lock`
2. Enqueue the element to the `buf`. It stores a **copy** of the element in the `buf`
3. Release the `lock`

# Send on a Full Channel

Actions when [`G`](Go%20Goroutines%20and%20Scheduler%20Internals.md) sends on a full channel (blocking call):

1. Acquire the `lock`
2. Create a `sudog` for ourselves with element we are waiting to send and enqueue it to the `sendq`. A receiver will use it to resume us in the future
3. Make a `gopark` call to the [scheduler](Go%20Goroutines%20and%20Scheduler%20Internals.md) and release the `lock`
4. Scheduler changes `G` state from running to waiting and removes association between `G` and [`M`](Go%20Goroutines%20and%20Scheduler%20Internals.md)
5. Scheduler dequeues the runnable `G'` from the [run queue](Go%20Goroutines%20and%20Scheduler%20Internals.md) and schedules it onto `M`

Essentially a user-level [context switch](Context%20Switch.md), we blocked the `G` but not the OS thread `M` that started running a `G'`

# Send on an Empty Channel

Actions when [`G`](Go%20Goroutines%20and%20Scheduler%20Internals.md) sends on an empty channel and there is a blocked receiver `G'`:

1. Acquire the `lock`
2. Dequeue the `sudog` struct of the waiting receiver `G'` from the `recvq`
3. Directly copy the element to the `element` field of the waiting receiver `G'` `sudog`. An optimization, when the blocked goroutine `G'` resumes it doesn't have to acquire the lock and dequeue the element from `buf`
4. Make a `goready(G')` call to the [scheduler](Go%20Goroutines%20and%20Scheduler%20Internals.md) and release the `lock`
5. Scheduler changes `G'` state from waiting to runnable and enqueues it to the [run queue](Go%20Goroutines%20and%20Scheduler%20Internals.md). It will be eventually scheduled since it's on the run queue
6. `G` continues executing

# Receive from a Channel

Actions when [`G`](Go%20Goroutines%20and%20Scheduler%20Internals.md) receives from a non-empty and non-full channel:

1. Acquire the `lock`
2. Dequeue the element from the `buf`. It **copies** the element in `buf` to variable in assignment
3. Release the `lock`

# Receive from an Empty Channel

Actions when [`G`](Go%20Goroutines%20and%20Scheduler%20Internals.md) receives from an empty channel (blocking call):

1. Acquire the `lock`
2. Create a `sudog` for ourselves with address we are waiting to receive to and enqueue it to the `recvq`. A sender will use it to resume us in the future
3. Make a `gopark` call to the [scheduler](Go%20Goroutines%20and%20Scheduler%20Internals.md) and release the `lock`
4. Scheduler changes `G` state from running to waiting and removes association between `G` and [`M`](Go%20Goroutines%20and%20Scheduler%20Internals.md)
5. Scheduler dequeues the runnable `G'` from the [run queue](Go%20Goroutines%20and%20Scheduler%20Internals.md) and schedules it onto `M`

# Receive from a Full Channel

Actions when [`G`](Go%20Goroutines%20and%20Scheduler%20Internals.md) receives from a full channel and there is a blocked sender `G'`:

1. Acquire the `lock`
2. Dequeue the element from the `buf`
3. Dequeue the `sudog` struct of the waiting sender `G'` from the `sendq`
4. Enqueue the element `elem` from the `sudog` into the `buf`. An optimization, when the blocked goroutine `G'` resumes it doesn't have to acquire the lock to enqueue the element
5. Make a `goready(G')` call to the [scheduler](Go%20Goroutines%20and%20Scheduler%20Internals.md) and release the `lock`
6. Scheduler changes `G'` state from waiting to runnable and enqueues it to the [run queue](Go%20Goroutines%20and%20Scheduler%20Internals.md). It will be eventually scheduled since it's on the run queue
7. `G` continues executing

# Resume in unbuffered/synchronous Channels

Unbuffered channels always work as direct send case

1. Receiver waiting → sender directly writes to receiver's stack from `sudog`
2. Sender waiting → receiver directly writes to sender's stack from `sudog`

# Select Statement

1. All channels are locked
2. A `sudog` is put in the`sendq`/`recvq` queues of all channels
3. Channels unlocked, all the selecting G is paused
4. [CAS](Atomic%20Instructions.md) operation so there is only one winning case
5. Resuming mirrors the pause sequence

# References

- [Source code: chan.go](https://github.com/golang/go/blob/master/src/runtime/chan.go)
- [Source code: runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go)
- [Source code: proc.go](https://github.com/golang/go/blob/master/src/runtime/proc.go)
- [go/src/runtime/HACKING.md at master · golang/go · GitHub](https://github.com/golang/go/blob/master/src/runtime/HACKING.md)
