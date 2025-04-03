---
tags:
  - Distributed-Systems
  - DB
---

# Overview

The transactional outbox pattern solves the dual-write problem in distributed systems, ensuring state consistency when a single operation involves updating a database and sending a message or event notification

Traditional [distributed transactions (2PC)](Two-Phase%20Commit.md) are often impractical due to their performance overhead. Additionally, many databases and message brokers do not support 2PC, and even when they do, coupling the service to both systems is generally undesirable.

# Implementation

1. Application writes the event to the outbox table within the [same transaction](ACID.md) as the business data update
2. After the transaction commits, a separate process (message relay) reads events from the outbox and reliably publishes them to the message bus, ensuring [eventual consistency](Consistency%20Models.md)
3. After publishing the event, it is deleted from the outbox or marked as processed to prevent duplicates

There are a couple of different ways to implement a message relay:

## Polling

Message relay publishes messages by periodically polling the outbox table

To prevent multiple relays from reading the same events, we use `SELECT FOR UPDATE` to lock the rows until the transaction is committed. To avoid waiting for other transactions to commit, we use `SKIP LOCKED`, which skips any rows that cannot be immediately locked

Once the transaction is committed and the lock is released, other relays can select the processed records. To prevent this, we delete the processed rows from the outbox table within the transaction

```sql
BEGIN;

DELETE FROM outbox
WHERE id IN (
	SELECT o.id FOR UPDATE SKIP LOCKED
	FROM outbox o
	ORDER BY id
	LIMIT 10
) RETURNING *;

-- publish messages (batch) here

COMMIT;
```

## Change Data Capture (CDC)

Message relay can use [Change Data Capture (CDC)](CDC) to tail the events from the outbox table

# References

- [Transactional outbox pattern - AWS Prescriptive Guidance](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html)
- [Transactional Outbox Pattern - gmhafiz Site](https://www.gmhafiz.com/blog/transactional-outbox-pattern/)
- [Microservices Patterns: With Examples in Java. Chris Richardson](References.md#Microservices%20Patterns%20With%20Examples%20in%20Java.%20Chris%20Richardson)
- [Pattern: Transactional outbox](https://microservices.io/patterns/data/transactional-outbox.html)
- [Mastering Data Consistency: A Deep Dive into the Transactional Outbox Pattern](https://blog.swcode.io/microservices/2023/10/17/transactional-outbox/)
