---
tags:
  - Distributed-Systems
---

# Consistency

Cluster uses asynchronous replication and a last failover wins merge function. This means that the last elected master dataset eventually replaces all replicas

A master is failed over only if it's unreachable by the majority of masters for at least `NODE_TIMEOUT`:

- If the partition is fixed before that time, no writes are lost
- If the partition lasts longer, writes performed on the minority side up to that point may be lost. However, the minority side stops accepting writes after `NODE_TIMEOUT` time has elapsed without contact with the majority, ensuring no further writes are lost

# Availability

Redis Cluster survives partitions where the majority of the master nodes are reachable and there is at least one reachable replica for every master that is no longer reachable

Redis Cluster is not available in the minority side of the partition. In the majority side it becomes available after `NODE_TIMEOUT` plus a few more seconds required for a replica to get elected and failover its master

# Merge Operations Are Avoided

The Redis Cluster avoids conflicting versions of the same key-value pair in multiple nodes, because merging complex data types, like Lists or Sorted Sets, can cause a performance bottleneck

[CRDTs](Conflict-Free%20Replicated%20Data%20Type.md) or [synchronously replicated state machines](Replication.md) can model Redis-like data types, but their runtime behavior differs from Redis Cluster

# Sharding Model

The key space is divided into 16384 hash slots, with each master node handling a subset of slots. When the cluster is stable, a single hash slot will be served by a single node

A hash slot is calculated using the following algorithm:

```
HASH_SLOT = CRC16(key) mod 16384
```

Redis Cluster supports multi-key operations as long as all of the keys belong to the same hash slot

A shard is defined as a collection of nodes that serve the same set of slots and that replicate from each other. A shard may only have a single master at a given time, but may have multiple or no replicas. It is possible for a shard to not be serving any slots while still having replicas

## Hash Tags

Hash tags are a way to ensure that multiple keys are allocated in the same hash slot

If the key contains a `{…}` pattern only the substring between `{` and `}` is hashed in order to obtain the hash slot

## MOVED Redirection

Clients can send queries to any node, including replicas. If the node serves the hash slot, it processes the query. Otherwise, it checks its internal hash slot to node map, and responds with a `MOVED` error. The error includes the hash slot and the correct node's address, requiring the client to resend the query to the provided address

## Resharding

During resharding, Redis Cluster moves hash slots (all the keys that hashes into this hash slot) from one node to another

### ASK Redirection

While `MOVED` indicates a permanent hash slot change to a new node, `ASK` means to send only the next query to the specified node

This is necessary because the next query for a hash slot might target a key still on the old node, so the client should query the old node before the new one

The semantics of `ASK` redirection from the point of view of the client are as follows:

- If `ASK` redirection is received, send only the query that was redirected to the specified node but continue sending subsequent queries to the old node
- Start the redirected query with the `ASKING` command
- Don't yet update local client tables to map the hash slot to the new node
- Once hash slot migration is completed, the old node sends a `MOVED` error, and the client permanently maps hash slot to the new node

### Multi-key Operations

During a resharding the multi-key operations targeting keys that all exist and all still hash to the same slot are still available

Operations on keys that don't exist or split between the source and destination nodes, will generate a `TRYAGAIN` error

# Node-to-node Communication

Redis Cluster is a full mesh where every node is connected with every other node using a TCP connection

While nodes form a full mesh, nodes use a [gossip protocol](Gossip%20Protocol.md) to avoid exchanging too many messages between nodes, so the number of messages is not exponential

Every node maintains the following information about other nodes: 

- The node ID 
- IP and port of the node
- What is the master of the replica node
- Last time the node was pinged and the last time the pong was received
- The set of hash slots served

# Scaling Reads Using Replica Nodes

Replica nodes usually redirect clients to the master, but clients can scale reads using the [`READONLY`](https://redis.io/commands/readonly) command. This configures connection into readonly mode and allow replicas to serve read requests with possibly stale data, making write queries unsupported

# References

- [Redis cluster specification \| Docs](https://redis.io/docs/latest/operate/oss_and_stack/reference/cluster-spec/#cluster-topology)
- [Scale with Redis Cluster \| Docs](https://redis.io/docs/latest/operate/oss_and_stack/management/scaling/)
- [Scaling a High-traffic Rate Limiting Stack with Redis Cluster — brandur.org](https://brandur.org/redis-cluster)
