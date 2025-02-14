---
tags:
  - Distributed-Systems
  - TODO
---

# Problem Definition

#TODO: [21: Distributed Locking \| Systems Design Interview Questions With Ex-Google SWE - YouTube](https://youtu.be/Lp8oITg0MiI?si=seUbLdBLFTY6Ullo)

A distributed lock that multiple nodes can use to ensure mutual exclusion

It must be fault tolerant and highly available, meaning the lock service must have multiple nodes, otherwise if the only node dies, the lock service would be unavailable. So the state of the lock must be replicated between multiple nodes

It must be [strongly consistent](Consistency%20Models.md), meaning we can't use [asynchronous replication](Replication.md) between leader and followers (which only ensures [eventual consistency](Consistency%20Models.md)), because a client might grab a lock from the follower event though the lock is already y held, but leader just hadn't replicated to the follower yet

Locks must support expiration, otherwise a dead client holding the lock would cause the lock to be held indefinetly

---

# Overview

Implementing a distributed lock requires care: even if a node believes that it holds the lock, that doesn't necessarily mean a quorum of nodes agrees. A node may have formerly been the leader, but if the other nodes declared it dead in the meantime (e.g., due to a network interruption or GC pause), it may have been demoted and another leader may have already been elected. If a node continues acting as the holder of the lock, even though the majority of nodes have declared it dead, it can lead to conflicts

Let's see an example:

![distributed lock incorrect|500](distributed%20lock%20incorrect.png)

If a client holding a lease is paused for too long, its lease expires. Another client can then acquire the lease and begin updating the resource. When the paused client comes back, it believes (incorrectly) that it still has a valid lease and also updates the resource, leading to conflicting updates and potential data corruption

# Fencing Tokens

When implementing the distributed lock, we must ensure that a node that is under a false belief of being the holder of the lock cannot disrupt the rest of the system

To achieve this, we can use fencing tokens:

![distributed lock fencing tokens|500](distributed%20lock%20fencing%20tokens.png)

Every time the lock server grants a lease, it also returns a fencing token, which is a number that increases every time a lock is granted (e.g., incremented by the lock service). We require that every time a client sends an update to the resource, it includes its fencing token

The resource itself to take an active role in checking tokens by rejecting any writes with an older token than one that has already been processed—it is not sufficient to rely on clients checking their lock status themselves

# References

- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann)
- [21: Distributed Locking \| Systems Design Interview Questions With Ex-Google SWE - YouTube](https://youtu.be/Lp8oITg0MiI?si=seUbLdBLFTY6Ullo)
- [How to do distributed locking — Martin Kleppmann’s blog](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
- [Is Redlock safe? - \<antirez\>](https://antirez.com/news/101)
- [SET \| Docs](https://redis.io/docs/latest/commands/set/)
