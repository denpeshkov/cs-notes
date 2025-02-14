---
tags:
  - Go
  - TODO
---

# Internals

![Go scheduler|550](Go%20scheduler.png)

	

## The `runtime.g` Struct

``` go
type g struct {
	// Stack describes the actual stack memory: [stack.lo, stack.hi).
	stack stack
	// stackguard0 is the stack pointer compared in the Go stack growth prologue.
	// It is stack.lo+StackGuard normally, but can be StackPreempt to trigger a preemption.
	stackguard0 uintptr
	// stackguard1 is the stack pointer compared in the //go:systemstack stack growth prologue.
	// It is stack.lo+StackGuard on g0 and gsignal stacks.
	// It is ~0 on other goroutine stacks, to trigger a call to morestackc (and crash).
	stackguard1 uintptr
	
	m *m // current m
	
	gopc uintptr // pc of go statement that created this goroutine
	startpc uintptr // pc of goroutine function to execute
	...
}

// Stack describes a Go execution stack.
// The bounds of the stack are exactly [lo, hi), with no implicit data structures on either side.
type stack struct {
	lo uintptr
	hi uintptr
}
```

Represents a goroutine. When a goroutine exits, its `g` object is returned to a pool of free `g`s and can later be reused for some other goroutine. It contains the fields necessary to keep track of its stack and current status. It also contains reference to the code that it is responsible for running

## The `runtime.m` Struct

```go
type m struct {
	g0 *g // goroutine with scheduling stack
	
	gsignal *g // signal-handling g
	goSigStack gsignalStack // Go-allocated signal handling stack
	spinning bool // m is out of work and is actively looking for work
	
	curg *g // current running goroutine
	p puintptr // attached p for executing go code (nil if not executing go code)
	nextp puintptr
	oldp puintptr // the p that was attached before executing a syscall
	...
}
```

Represents an OS thread (machine). It has pointers to fields such as the `g` that it is currently running and associated `p`

When `m` executes code, it has an associated `p`. When `m` is idle or in a [system call](#Handoff%20and%20System%20Calls), it does not need a `p`

## Spinning

## The `runtime.p` Struct

```go
type p struct {
	m muintptr // back-link to associated m (nil if idle)
	mcache *mcache // cache for small objects
	pcache pageCache // cache of pages the allocator can allocate from without a lock
	
	// Local runnable queue of goroutines. Accessed without lock.
	runqhead uint32
	runqtail uint32
	runq [256]guintptr
	// A runnable G that was ready'd by the current G and should be run next instead of what's in
	// runq if there's time remaining in the running G's time slice.
	// It will inherit the time left in the current time slice.
	runnext guintptr
	
	// Cache of mspan objects from the heap.
	mspancache struct {
		len int
		buf [128]*mspan
	}
	
	// Cache of a single pinner object to reduce allocations from repeated pinner creation.
	pinnerCache *pinner
	
	// Timer heap.
	timers timers
	...
}

// Per-thread (in Go, per-P) cache for small objects.
// This includes a small object cache and local allocation stats.
type mcache struct {
	_ sys.NotInHeap

	nextSample uintptr // trigger heap sample after allocating this many bytes
	scanAlloc  uintptr // bytes of scannable heap allocated

	// Allocator cache for tiny objects w/o pointers.
	tiny       uintptr
	tinyoffset uintptr
	tinyAllocs uintptr
	alloc [numSpanClasses]*mspan // spans to allocate from, indexed by spanClass
	stackcache [_NumStackOrders]stackfreelist // small stach cache
	flushGen atomic.Uint32
}
```

Represents a processor abstraction, resources required to execute a `g`. Contains a [lock-free](Non-blocking%20Data%20Structures.md) **local run-queue** `p.runq` and [caches](Go%20Memory%20Allocator.md)

There are exactly `GOMAXPROCS` `p`s, thus even if we had a lot of `m`s, there are no scalability issues with [work stealing](#Work%20Stealing) and per `p` [caches](Go%20Memory%20Allocator.md) accesses

## The `runtime.schedt` Struct

```go
type schedt struct {
	lock mutex
	
	midle muintptr // idle m's waiting for work
	
	pidle puintptr // idle p's
	
	nmspinning atomic.Int32
	
	// Global runnable queue.
	runq     gQueue
	runqsize int32
	
	// Central cache of sudog structs.
	sudoglock  mutex
	sudogcache *sudog
	
	// sysmonlock protects sysmon's actions on the runtime.
	sysmonlock mutex
}
```

Represents a scheduler itself. Contains a **global run-queue** `schedt.runq` protected by a [mutex](Mutex.md) `schedt.lock`

# Scheduling Algorithm

```go

```

# Handoff and System Calls

If OS thread `m` itself is blocked, e.g. in a [system call](System%20Calls.md), the scheduler has to handoff `p` to other `m` in order to prevent CPU underutilization

However, not all system calls are blocking in practice, e.g. [vDSO](System%20Calls.md). Having goroutines go through the full work of releasing their current `p` and then re-acquiring one for these system calls has two problems:

1. There's a bunch of overhead involved in locking (and releasing) all of the data structures involved
2. If there's more runnable `g`s than `p`s, a `g` that makes a syscall won't be able to re-acquire a `p` and will have to park itself; the moment it released the `p`, something else was scheduled onto it

So the scheduler has two ways of handling blocking system calls, a pessimistic way for system calls that are expected to be slow; an optimistic way for ones that are expected to be fast

The pessimistic system call path implements the approach where `p` is handed off `m` before the system call

The optimistic system call path doesn't release the `p`; instead, it sets a special `p` state flag and just makes the system call. The `sysmon` goroutine periodically looks for `p`s that have been in this state for too long and hands them off `m`

When `m` returns from a syscall the scheduler will check if the old `p` is available. If it is, the `m` will associate itself with it. If it is not, `m` will associate itself with any idle `p`. And if there are no idle `p`s, `g` will be dissociated from `m` and added to the `schedt.runq`

Thread `m` itself becomes idle, so we won't run more `g`s than our target parallelism level

# Fairness

The scheduler uses following mechanisms to provide fairness at minimal cost. Fairness helps preventing goroutines livelocks and starvation

## Goroutine Preemption

Scheduler uses a 10ms time slice after which it preempts the current goroutine, enqueues it to global `schedt.runq` and schedules another goroutine

To preempt a `g`, a scheduler spoofs `g`s stack limit by setting `g.stackguard0` to `stackPreempt` which causes stack overflow in function [prologue check code](#Goroutine%20Stack). At runtime (already in slow path) scheduler recognizes that actually there is enough stack and it was a preemption signal. This spoof technique is an optimization to reduce the overhead of checking the preemption status on each function invocation explicitly

#todo this allows preemptable scheduler instead of cooperative scheduler used before, where `g`s could be preempted only in specific blocking cases (for example, channel send or receive, I/O, waiting to acquire a mutex)

### Loop Preemption

Above approach doesn't work for loops because, there are no function calls, hence no `runtime.stachmap` metadata needed for [Go Garbage Collector](Go%20Garbage%20Collector.md)

Go uses signal-based preemption with conservative innermost frame scanning to preempt tight loops:

1. Send [OS signal](Interrupts%20and%20Exceptions.md) to preempt the goroutine
2. [Conservatively scan](Go%20Garbage%20Collector.md) the function stack: assume that anything that looks like a pointer is a pointer. Only needed for innermost frame
3. Precisely scan the rest of the stack

## Local Run Queue Design

To fairly schedule `g`s we need FIFO, because LIFO would mean that two `g`s constantly respawning each other would starve other `g`s. We would always pop this two `g`s from the top never reaching the bottom

On the other hand, from CPU perspective we need LIFO, because it provides better [cache locality](Memory%20Hierarchy%20and%20Locality.md). When `g1` writes data to memory and then `g2` resumes, using FIFO may result in `g2` finding that the data is no longer in the cache, while LIFO ensures that data will still be in a cache

To provide this LIFO/FIFO behavior the scheduler saves a runnable `g'`, that was ready'd by the current `g` and should be run next instead of what's in `p.runq`, into the one element LIFO slot `p.runnext`. To ensure `g'` is not stolen by other `m`, goroutines from `p.runnext` are not allowed to be stolen for 3us

However, this `p.runnext` LIFO behavior can cause starvation. In this code, `g1` and `g2` always respawn each other in `p.runnext` never checking `p.runq`:

```go
c1, c2 := make(chan int), make(chan int)

// g1
go func() {
	for {
		c1 <- 1
		<-c2
	}
}()

// g2
go func() {
	for {
		<-c1
		c2 <- 2
	}
}()
```

To prevent `p.runq` starvation, `g` taken from `p.runnext` inherits current time slice and will be preempted within 10ms

## Global Run Queue Polling

To avoid `schedt.runq` starvation, scheduler checks it every $1/61$ scheduling tick

## Network Poller Async Notifications

To avoid [network poller](Go%20Network%20Poller.md) starvation, network poller uses background thread to `epoll` the network if it wasn't polled by the main worker thread

Polling used for global run queue is not applicable here, because checking the network involves system calls like `epoll` which adds an overhead to the scheduler fast path

# Goroutines Stack

#todo See: [I don't know where goroutine stacks are allocated in virtual memory : r/golang](https://www.reddit.com/r/golang/comments/1d5jgcd/i_dont_know_where_goroutine_stacks_are_allocated/)

A [traditional paging based stacks](Call%20Stack.md) can't be used:

- Not enough address space to support millions of 1Gb stacks. Because we need to reserve the virtual memory for all the stacks at the program startup, and we have 48 bits address space on 64 bit CPU, we can support only 128000 stacks
- Suboptimal granularity due to minimum page size of 4Kb
- Performance overhead due to system calls used to release unused pages

Instead, Go uses growable stacks allocated on the [heap](Heap%20Memory.md), which allows a goroutine to start with a single 2Kb stack which grows and shrinks as needed. A compiler inserts a prologue at the start of each function, which checks for the overflow by comparing the current [stack pointer](Call%20Stack.md) to `g.stackguard0`. When a stack overflow check triggers, runtime calls `runtime⋅morestack` to grow a stack:

- Allocate a new stack 2x the size of the old stack. This amortizes the cost of copying
- Copy the old stack to the new stack
- Adjust all pointers on the stack by utilizing [`runtime.stackmap`](Go%20Garbage%20Collector.md) metadata
- Adjust all pointers into the stack from outside. There are a few despite escape analysis
- Deallocate the old stack

The `p.mcache.stackcache` is a cache of small stacks

# References

- [Source code: proc.go](https://github.com/golang/go/blob/master/src/runtime/proc.go)
- [Source code: runtime2.go](https://github.com/golang/go/blob/master/src/runtime/runtime2.go#L551)
- [Source code: stack.go](https://github.com/golang/go/blob/master/src/runtime/stack.go)
- [GopherCon 2018: Kavya Joshi - The Scheduler Saga - YouTube](https://youtu.be/YHRO5WQGh0k?si=ZHiNxNxG1GAOff9D)
- [Dmitry Vyukov — Go scheduler: Implementing language with lightweight concurrency - YouTube](https://youtu.be/-K11rY57K7k?si=goeOdKxqd-cXBZE1)
- [runtime: tight loops should be preemptible · Issue #10958 · golang/go · GitHub](https://github.com/golang/go/issues/10958)
- [dotGo 2017 - JBD - Go's work stealing scheduler - YouTube](https://youtu.be/Yx6FBsGNOp4?si=3Onn5gRHTaPvlbtb)
- [Go's work-stealing scheduler · rakyll.org](https://rakyll.org/scheduler/)
- [Scheduling In Go : Part I - OS Scheduler](https://www.ardanlabs.com/blog/2018/08/scheduling-in-go-part1.html)
- [The Golang Scheduler](https://www.kelche.co/blog/go/golang-scheduling/)
- [Внутреннее устройство планировщика Go // Демо-занятие курса «Golang Developer. Professional» - YouTube](https://www.youtube.com/watch?v=uU0FbA3u5vI)
- [ТиПМС 9. Планировщик Go - YouTube](https://www.youtube.com/watch?v=TRoFIaHdjB8&t=4001s) #todo
- [Contiguous Stacks Design Doc - Google Docs](https://docs.google.com/document/d/1wAaf1rYoM4S4gtnPh0zOlGzWtrZFQ5suE8qr2sD8uWQ/pub) #todo
- [Go Preemptive Scheduler Design Doc - Google Docs](https://docs.google.com/document/d/1ETuA2IOmnaQ4j81AtTGT40Y4_Jr6_IDASEKg0t0dBR8/edit#heading=h.3pilqarbrc9h)
- [Scalable Go Scheduler Design Doc - Google Docs](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw/edit)
- [HACKING.md](https://github.com/golang/go/blob/master/src/runtime/HACKING.md)
- [GopherCon 2020: Austin Clements - Pardon the Interruption: Loop Preemption in Go 1.14 - YouTube](https://www.youtube.com/watch?v=1I1WmeSjRSw) #TODO 
- [Chris's Wiki :: blog/programming/GoSchedulerAndSyscalls](https://utcc.utoronto.ca/~cks/space/blog/programming/GoSchedulerAndSyscalls)
- [100 Go Mistakes and How to Avoid Them. Teiva Harsanyi](References.md#100%20Go%20Mistakes%20and%20How%20to%20Avoid%20Them.%20Teiva%20Harsanyi)
