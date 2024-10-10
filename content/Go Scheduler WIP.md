When `g` yields it's enqueued to global `schedt.runq` because if we yield we are not highly prioritized (`globrunqput`)

When `g` readies another `g'`, `g'` is enqueued to local `p.runq` **without locking** (`runqput`) (see [Fairness](#Fairness))  
If local `p.runq` is full, we enqueue **batch** (half of `g`s) to global `schedt.runq`, because it means that we have to much work on our `p`. Note that in that case we **need to hold the lock** `schedt.lock` for global `schedt.runq` (contended slow case)

Function `schedule` schedules a goroutine. It calls `findRunnable` to find next `g` to run:

1. Call `runqget` to get `g` from `p.runq` and run it
2. Call `globrunqget`. It gets `g` from global `schedt.runq` acquiring the lock `schedt.lock` (contended slow case) and also moves the **batch** of `g`s from `schedt.runq` to `p.runq` to minimize the # of accesses to `schedt.runq`
3. [Poll network](Go%20Network%20Poller.md)
4. Steal **batch** (half of `g`s) from others (random) `p`s by calling `runqsteal` (work stealing) 
5. Sleep

Local `p.runq` have a single producer but multiple consumers, because there can be multiple `p`s that steal work from our `p.runq`. Thus, when enqueueing to `p.runq` we can use [atomic store](Atomic%20Instructions.md  ) but when dequeueing we need to use [spinlocks](Spinlock.md) ([CAS](Atomic%20Instructions.md))

Because `findRunnable` would starve `schedt.runq` if `p.runq` is always non empty, `schedule` tries to dequeue one `g` from `schedt.runq` in $1/61$ iteration of scheduling
