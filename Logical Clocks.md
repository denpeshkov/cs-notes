---
tags:
  - Distributed-Systems
---

# Overview

Timestamps from physical clocks can be inconsistent with [causality](Causality%20and%20Happens-Before.md), even if those clocks are synchronized using NTP

In contrast, logical clocks are designed to capture [causal dependencies](Causality%20and%20Happens-Before.md) by counting the number of events occurred

# Lamport Timestamps

- Each node maintains a timestamp (counter): $t$
- On any event occurring at the node: $t=t+1$
- On sending the message $m$: $t=t+1$; $send(t,m)$
- On receiving $(t',m)$: $t=max(t,t')+1$

The algorithm assumes a [crash-stop model](Models%20of%20Distributed%20Systems.md) (or a [crash-recovery model](Models%20of%20Distributed%20Systems.md) if the timestamp is maintained in a permanent storage)

Let $L(e)$ be the value of $t$ after the event $e$

- If $a \to b$ then $L(a) < L(b)$
- However, $L(a) < L(b)$ does not imply $a \to b$; Possible that $L(a)=L(b)$ for $a \ne b$

## Ordering

Using Lamport timestamps we can extend the happens-before partial order into a *total order*

We use the lexicographic order over (timestamp, node name) pairs: that is, we first compare the timestamps, and if they are the same, we break ties by comparing the node names:

- Let $N(e)$ be the name of the node at which event $e$ occurred. Then the pair $(L(e), N(e))$ uniquely identifies event $e$
- Define a *total order* $\prec$ using Lamport timestamps: $(a \prec b) \iff (L(a)<L(b) \lor (L(a)=L(b) \land N(a)<N(b)))$

Properties:

- This relation $\prec$ puts all events into a linear order: for any two events $a \ne b$ we have either $a \prec b$ or $b \prec a$
- It is a causal order: $(a \to b) \implies (a \prec b)$
- Given Lamport timestamps $L(a)$ and $L(b)$ with $L(A)<L(b)$ we can't tell whether $a \to b$ or $a \parallel b$; That is, we can't tell whether two events are concurrent

# Vector Clocks

Vector clocks allow us to detect when events are concurrent. While Lamport timestamps are just a single integer (possibly with a node name attached), vector timestamps are a list of integers, one for each node in the system

- Assume $n$ nodes in the system: $N=\langle N_1,N_2, \dots ,N_n\rangle$
- Each node $N_i$ maintains a vector timestamp $T$ of size $n$; Initially all zeros: $T=\langle 0,0,\dots ,0\rangle$
- On any event occurring at node $N_i$: $T[i]=T[i]+1$
- On sending the message $m$ at node $N_i$: $T[i]=T[i]+1$; $send(T,m)$
- On receiving $(T',m)$ at node $N_i$: $T[j]=max(T[j],T'[j])$ for every $j \in \set{1,\dots ,n}$; $T[i]=T[i]+1$

## Ordering

Define the following order on vector timestamps in the system with $n$ nodes:

- $T=T'$ iff $T[i]=T'[i]$ for all $i \in \set{1,\dots n}$
- $T\leq T'$ iff $T[i]\leq T'[i]$ for all $i \in \set{1,\dots n}$
- $T<T'$ iff $T\leq T'$ and $T\ne T'$
- $T\parallel T'$ iff $T\nleq T'$ and $T'\nleq T$

Define vector timestamp of event $a$ as $V(a)=\langle t_1,\dots ,t_n\rangle$

$V(a)\leq V(b)$ iff $(\{a\}\cup \set{e|e\to a})\subseteq (\{b\}\cup \set{e|e\to b})$

- $(V(a)<V(b)) \iff (a\to b)$
- $(V(a)=V(b)) \iff (a=b)$
- $(V(a)\parallel V(b)) \iff (a\parallel b)$

The partial order over vector timestamps corresponds exactly to the partial order defined by the [happens-before](Causality%20and%20Happens-Before.md) relation

# Dotted Version Vectors

#TODO 

# References

- [Distributed Systems Notes. Martin Kleppmann](References.md#Distributed%20Systems%20Notes.%20Martin%20Kleppmann)
- [Distributed Systems 4.1: Logical time - YouTube](https://youtu.be/x-D8iFU1d-o?si=orzuqBqQsNAfxn6M)
- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann) #TODO 
- [Understanding Distributed Systems (2nd ed). Roberto Vitillo](References.md#Understanding%20Distributed%20Systems%20(2nd%20ed).%20Roberto%20Vitillo) #TODO 
- [Distributed Systems 5.1: Replication - YouTube](https://youtu.be/mBUCF1WGI_I?si=yjtGayUPheUGZERm) #TODO 
- [CSE138 (Distributed Systems) L2: time and clocks, causality and happens-before, network models - YouTube](https://youtu.be/3J9lqpMjA2I?si=iqzOg9JucHDO03sk)
