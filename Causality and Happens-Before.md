---
tags:
  - Distributed-Systems
  - TODO
---

# Happens-Before Relation

We assume that there is a *strict total order* on the events that occur at the same node

Event $a$ happens-before event $b$ (written $a \to b$) iff:

- $a$ and $b$ occurred at the same node, and $a$ occurred before $b$ in that node's execution order
- event $a$ is the sending of some message $m$, and event $b$ is the receipt of that same message $m$
- there exists an event $c$ such that $a \to c$ and $c \to b$

The happens-before relation is a *irreflexive partial order*: it is possible that neither $a \to b$ nor $b \to a$. In that case, $a$ and $b$ are concurrent $a\parallel b$

# References

- [Distributed Systems Notes. Martin Kleppmann](References.md#Distributed%20Systems%20Notes.%20Martin%20Kleppmann)
- [Distributed Systems 4.1: Logical time - YouTube](https://youtu.be/x-D8iFU1d-o?si=NES6wXImu9M631-K)
- [Distributed Systems 3.3: Causality and happens-before - YouTube](https://youtu.be/OKHIdpOAxto?si=rrCkm_SUMNsggTpJ)
- [Distributed Systems 4.1: Logical time - YouTube](https://youtu.be/x-D8iFU1d-o?si=4jKJJJuZ8NhMI9u5)
- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann)
- [CSE138 (Distributed Systems) L2: time and clocks, causality and happens-before, network models - YouTube](https://youtu.be/3J9lqpMjA2I?si=iqzOg9JucHDO03sk)
