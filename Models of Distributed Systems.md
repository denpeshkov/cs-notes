---
tags:
  - Distributed-Systems
---

# Network Models

Assuming bidirectional point-to-point communication between two nodes:

- **Reliable links**: A message is received if and only if it is sent. Messages may be reordered
- **Fair-loss links**: Messages may be lost, duplicated, or reordered. Repeated attempts will eventually deliver the message
- **Arbitrary links**: A malicious adversary may interfere with messages (eavesdrop, modify, drop, spoof, replay)

A fair-loss link can be turned into a reliable link by retrying on the sender's side and deduplicating on the receiver's side

An arbitrary link can be transformed into a fair-loss link, assuming the adversary does not block communication indefinitely, by using cryptographic techniques such as TLS

# Timing Models

- **Synchronous**: Assumes bounded network delay, process pauses, and clock error. This does not imply exactly synchronized clocks or zero network delay; only that these factors will not exceed some fixed upper bound
- **Partially synchronous**: Means that a system behaves like a synchronous system most of the time, but it sometimes exceeds the bounds for network delay, process pauses, and clock drift
- **Asynchronous**: An algorithm is not allowed to make any timing assumptions. The system does not rely on timeouts and lacks a clock

# Nodes Models

- **Crash-stop faults**: A node may fail by crashing, and stops responding permanently
- **Crash-recovery faults**: Nodes may crash and later recover. Nodes have persistent storage that survives crashes, but their in-memory state is lost
- **Byzantine faults**: Nodes may behave arbitrarily, including attempting to deceive or mislead other nodes

# References

- [Distributed Systems Notes. Martin Kleppmann](References.md#Distributed%20Systems%20Notes.%20Martin%20Kleppmann)
- [Distributed Systems 2.3: System models - YouTube](https://youtu.be/y8f7ZG_UnGI?si=TZYVONteUfxQT3mw)
- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann)
