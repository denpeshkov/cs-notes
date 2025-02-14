---
tags:
  - Distributed-Systems
  - TODO
---

# Cluster

Kafka cluster is a group of Kafka brokers. All brokers within a cluster share meta information of other brokers in the same cluster

# Brokers

A single Kafka server is called a broker. Each broker hosts some set of partitions and handles incoming requests to write new events to those partitions or read events from them. Brokers also handle replication of partitions between each other

Kafka brokers operate as part of a cluster. Within a cluster of brokers, one broker functions as the cluster controller and is elected automatically from the live members of the cluster. The controller is responsible for administrative operations, including assigning partitions to brokers and monitoring for broker failures

# Topics

A topic is a collection of related messages. Unlike in messaging queues, consuming messages does not delete them from topics. This allows messages to be (re)read multiple times by multiple consumers

# Partitions

To enable horizontal scaling, a topic is divided into one or more partitions, each of which can reside on a separate broker. A partition is an append-only log that maintains strict message ordering

Internally, a partition is further split into segments, which are stored as `.log` files on the broker's disk. When a segment reaches its size limit, the broker closes the file and starts a new one

Old log segments in Kafka are deleted based on a configurable retention policy. When the retention period for a Kafka topic expires, Kafka identifies and deletes entire segments that fall outside the retention window. This approach ensures that Kafka can efficiently reclaim disk space by removing entire segments rather than individual messages

Partitions serve as the primary storage unit for Kafka events and enable parallelism, allowing multiple events to be produced simultaneously across different partitions. Consumers can also distribute their workload by having individual instances read from various partitions

## Replication

Kafka replicates the event log for each topic's partitions across a configurable number of brokers to ensure fault tolerance. One broker is designated as the leader, while the others serve as followers

The leader handles all read and write operations, while followers replicate the leader's data

The logs on the followers are identical to the leader's log, all having the same offsets and messages in the same order. Although at any given time, the leader may have a few unreplicated messages at the end of its log

If the leader fails, a follower is automatically elected as the new leader

## Offsets

An offset is a unique numeric ID assigned to each message within a partition. Consumers use offsets to keep track of the events they have processed. This enables them to replay events by reading from a specific offset

Internally the offset represents the position of a message within the partition's log

# Producers

A Kafka producer sends messages to a topic, and messages are distributed to partitions based on a routing mechanism

For a message to be successfully written into a Kafka topic, a producer must specify a level of acknowledgment

Producers only write data to the current leader broker for a partition

The combination of topic, partition and offset uniquely identifies the message

## Routing

Messages are routed to partitions based on their key ([hash](Hash%20Map.md) of the key). Messages with the same key are always stored in the same partition and, because a partition is append-only, are always in order. As a result, message ordering is only guaranteed within individual partitions, not across the entire topic

If a message has no key, Kafka uses a round-robin mechanism to distribute messages across partitions

## Acknowledgment

The `acks` setting determines the number of acknowledgments the producer requires the leader to have received from replicas before considering a write successful:

1. `acks=0`: The producer does not wait for any acknowledgment from the server
2. `acks=1`: The leader will write the record to its local log but will respond without awaiting full acknowledgement from all replicas. If an acknowledgment is not received, the producer may retry the request
3. `acks=all`: The leader will wait for the full set of in-sync replicas to acknowledge the record

# Consumers

Consumers can read from one or more partitions at a time, the message order is not guaranteed across multiple partitions because they are consumed simultaneously, but the message read order is still guaranteed within each individual partition

Contrary to messaging queues, Consuming a message from the topic (partition) does not destroy it.

In a traditional message queue, you can often scale the number of consumers, but then you usually miss out on ordering guarantees altogether. In a traditional messaging topic, remember topic and queue being different things in a legacy messaging system, in a topic you keep ordering guarantees in place but then you sacrifice the ability to scale out consumers

## Consumer Groups

A consumer group is a collection of consumers that work together to consume messages from a set of topics

Each partition is consumed by exactly one consumer within the group. In the case of multiple groups, several consumers can read messages from the same partition as long as they are from different groups

Adding more consumers to a group increases the ability to consume messages in parallel, but the number of consumers is limited by the number of partitions. If there are more consumers than partitions, some consumers will remain idle

If a consumer in the group fails, the partitions it was responsible for are automatically reassigned to other consumers in the group

## Offset Committing

Each consumer in the group keeps track of its own position in each partition (its offset), allowing it to resume processing from the correct point after a restart or failure

The consumer commits offsets to an internal `__consumer_offsets` topic, which stores committed offset information for each topic-partition pair per consumer group ID

## Rebalancing

Rebalancing is the process by which Kafka redistributes partitions across consumers to ensure that each consumer is processing an approximately equal number of partitions

# Broadcasting

[Kafka: Broadcasting Messages Without Consumer Groups \| by Ravi Sharma \| Medium](https://medium.com/@ravisharma911993/kafka-broadcasting-messages-without-consumer-groups-5a374fcfb7bc) #TODO 

# References

- [Course | Apache Kafka Fundamentals - YouTube](https://www.youtube.com/playlist?list=PLa7VYi0yPIH2PelhRHoFR5iQgflg-y6JA)
- [Intro to Apache Kafka: How Kafka Works](https://www.confluent.io/blog/apache-kafka-intro-how-kafka-works/)
- [Apache Kafka Architecture Deep Dive: Introductory Concepts](https://developer.confluent.io/courses/architecture/get-started/)
- [Kafka architecture](https://www.redpanda.com/guides/kafka-architecture)
- [Designing Data-Intensive Applications: The Big Ideas Behind Reliable, Scalable, and Maintainable Systems. Martin Kleppmann](References.md#Designing%20Data-Intensive%20Applications%20The%20Big%20Ideas%20Behind%20Reliable,%20Scalable,%20and%20Maintainable%20Systems.%20Martin%20Kleppmann)
- [Kafka Fundamentals | Learn Apache Kafka with Conduktor](https://learn.conduktor.io/kafka/kafka-fundamentals/)
- [Kafka Advanced Concepts | Learn Apache Kafka with Conduktor](https://learn.conduktor.io/kafka/kafka-advanced-concepts/)
- [Kafka Deep Dive w/ a Ex-Meta Staff Engineer - YouTube](https://youtu.be/DU8o-OTeoCc?si=DIzBqvrYmAUCYLrL)
