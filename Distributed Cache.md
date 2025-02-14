---
tags:
  - Distributed-Systems
---

# Read Strategies

## Read-Through

- Service first makes a read request to the cache
- On a cache hit, the cached data is returned
- On a cache miss, the cache makes a read request to the DB and stores the data in the cache. Subsequent requests for this data will be cache hits

Advantages:

- The cache contains only requested data, which helps keep the cache size small
- As the service does not contact the DB, the implementation burden of DB requests is shifted to the cache

Disadvantages:

- The cached data may become stale if writes are made directly to the DB. To reduce stale data, we can set a TTL

## Cache-Aside

- Service first makes a read request to the cache
- On a cache hit, the cached data is returned
- On a cache miss, the service makes a read request to the DB. The cache is then populated with the data from the DB, so subsequent requests for this data will be cache hits

Advantages:

- The cache contains only requested data, which helps keep the cache size small

Disadvantages:

- Each cache miss results in three roundtrips
- The cached data may become stale if writes are made directly to the DB. To reduce the amount of stale data, we can set a TTL
- [Thundering Herd problem](#Cache%20Stampede%20(Thundering%20Herd)%20and%20Cache%20Avalanche) on cache misses when accessing the DB in case of a large number of services

# Write Strategies

## Write-Through

- Service first writes data to the cache
- Cache synchronously writes data to the DB

Advantages:

- Improved consistency between the cache and DB. However, inconsistencies may still occur, one solution is to use a [2PC](Two-Phase%20Commit.md)

Disadvantages:

- Slower writes since every write to the cache is synchronously written to the DB
- Most data is never read, so we incur unnecessary cost. We can configure a TTL to reduce wasted space

## Write-Back (Write-Behind)

- Service first writes data to the cache
- Cache asynchronously writes data to the DB

Advantages:

- Faster writes on average than write-through since writes to the DB are not blocking

Disadvantages:

- Most data is never read, so we incur unnecessary cost. We can configure a TTL to reduce wasted space
- Complex design because our cache must have high availability, so we cannot make tradeoffs against availability to improve performance or latency

## Write-Around

- Service writes data directly to the DB
- Cache is populated on cache misses using [Read-Through](#Read-Through) or [Cache-Aside](#Cache-Aside) strategies

# Thundering Herd (Cache Stampede) and Cache Avalanche

Thundering herd problem, also known as cache stampede, is a type of failure that occurs when a large number of read requests result in cache misses, causing a massive influx of traffic to the DB. This surge of requests can overload the DB, causing slowdowns or outages

Cache avalanche is a type of failure that happens when multiple or all cache entries expire simultaneously or within a short time window, leading to a sudden surge in requests to the DB

## Distributed Locks and Leases

To avoid multiple services querying the DB simultaneously, a service will attempt to acquire a [distributed lock (lease)](Distributed%20Lock.md) for the cache key when a cache miss occurs. Only the service that successfully acquires the lock will access the DB

It's important that a lock has an expiration time to avoid deadlocks, such locks are called leases

If the lock cannot be acquired, there are a few possible strategies:

- Services can wait until the value is recomputed and updated in the cache, for example using Pub/Sub for notifications
- A stale cache item can be retained and used by the services while the new value is being recomputed

Note that a true [distributed lock](Distributed%20Lock.md) ensuring strict mutual exclusion is not required. It's acceptable for multiple nodes to query the DB simultaneously — we just aim to minimize the number of such queries

### Implementation of a Lease in Redis

Since the goal is efficiency and avoiding the thundering herd problem, it is often unnecessary to incur the cost and complexity of a true [distributed lock](Distributed%20Lock.md) 

Service tries to acquire a lock using:

```
SET lock:<key> <UUID> NX EX <expiration_time>
```

- `NX`: Ensures the key is set only if it does not already exist.
- `EX <expiration_time>`: Automatically expires the lock to prevent deadlocks.

If the lock is acquired, the service queries the DB and populates the cache with the retrieved value. It then releases the lock using the following Lua script:

```lua
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
```

The Lua script ensures atomicity, and using the UUID as the key value, along with this script, implements a [fencing token](Distributed%20Lock.md)

Once the cache is updated, the service publishes the value to the Pub/Sub channel to notify waiting clients:

```
PUBLISH notif:<key> <value>
```

If the lock is not acquired, the service subscribes to the Pub/Sub channel and waits for the value:

```
SUBSCRIBE notif:<key>
```

To avoid waiting indefinitely for the notification, the service must implement a timeout, ensuring it doesn't deadlock if the publishing service dies

## Probabilistic Early Expiration

Each individual request may decide to proactively refresh the cache based on a probability that increases as the value nears expiration. Since the probabilistic decision is made independently by each process, the effect of the stampede is mitigated as fewer processes will expire at the same time

See [Optimal Probabilistic Cache Stampede Prevention](https://cseweb.ucsd.edu/~avattani/papers/cache_stampede.pdf) for an algorithm specification

## Staggered Expiration

For cache avalanche prevention, use staggered expiration by combining a base time-to-live (TTL) value with a random delta (jitter)

## Consistent Hashing

[Consistent hashing](Consistent%20Hashing.md) can be used to distribute cache entries across multiple cache servers evenly. It reduces the impact of cache avalanche and cache stampede by sharing the load among the servers

## Circuit Breakers and Rate Limiting

Implementing circuit breakers and [rate limiting](Rate%20Limiting.md) in the system can help prevent cache avalanche and cache stampede from escalating into more severe issues

## Request Collapsing

Request collapsing combines multiple requests for the same object into a single request, and then potentially using the response to satisfy all pending requests

Maintain a queue for requests targeting the same object. When a new request arrives while a fetch is in progress, it is added to the queue instead of triggering a separate fetch. Once the fetch completes, the response is cached and served to all queued requests simultaneously

This prevents multiple requests from querying the database at the same time. However, since request collapsing queues requests, it may sometimes increase response times

# Cache Penetration

Cache penetration happens when requested data is missing in both the cache and DB, causing requests to skip the cache and hit the DB. Repeated requests for nonexistent keys can lead to cache misses and overload the DB

## Null Values

Store a placeholder value, such as `null`, in the cache to represent nonexistent data. Subsequent requests for the same data will result in a cache hit, and the Service can handle `null` values. Set an appropriate TTL for these placeholder entries to prevent them from occupying cache space indefinitely

## Bloom Filter

When a record is added to storage, its key is also stored in a [Bloom filter](Bloom%20Filter.md). When fetching a record, the Service first checks the Bloom filter. If the key is not found, it doesn't exist in the cache or storage either, and the Service returns a `null` value. If the key is found, the Service checks the cache and storage. Since a Bloom filter can have false positives, some cache reads may still result in a miss

# References

- [Caching patterns - DB Caching Strategies Using Redis](https://docs.aws.amazon.com/whitepapers/latest/DB-caching-strategies-using-redis/caching-patterns.html)
- [AWS: Caching challenges and strategies](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)
- [Exploring Cache Data Consistency - Alibaba Cloud Community](https://www.alibabacloud.com/blog/600308?spm=a3c0i.27947514.3143942920.10.7ce246e9TM9UEL)
- [pdos.csail.mit.edu/6.824/papers/memcache-faq.txt](https://pdos.csail.mit.edu/6.824/papers/memcache-faq.txt)
- [Consistency between Redis Cache and SQL DB | Yunpeng's Blog](https://yunpengn.github.io/blog/2019/05/04/consistent-redis-sql/)
- [How Facebook served billions of requests per second Using Memcached](https://blog.bytebytego.com/p/how-facebook-served-billions-of-requests)
- [Thundering Herd/Cache Stampede – Distributed Computing Musings](https://distributed-computing-musings.com/2021/12/thundering-herd-cache-stampede/)
- [Thundering herd problem - Wikipedia](https://en.wikipedia.org/wiki/Thundering_herd_problem)
- [Lecture 17: Cache Consistency: Memcached at Facebook - YouTube](https://www.youtube.com/watch?v=eYZg0YJtFEE)
- [Lecture 16: Cache Consistency: Memcached at Facebook - YouTube](https://youtu.be/Myp8z0ybdzM?si=3Nn-01HA-_VUAxov)
- [NSDI '13 - Scaling Memcache at Facebook - YouTube](https://youtu.be/m4_7W4XzRgk?si=e_528Cs4SwlrJ86u)
- [Cache stampede - Wikipedia](https://en.wikipedia.org/wiki/Cache_stampede)
- [Caching Pitfalls Every Developer Should Know - YouTube](https://youtu.be/wh98s0XhMmQ?si=TUXUg4c8TIQ2WZDA)
- [A Crash Course in Caching - Final Part - by Alex Xu](https://blog.bytebytego.com/p/a-crash-course-in-caching-final-part)
- [Optimal Probabilistic Cache Stampede Prevention](https://cseweb.ucsd.edu/~avattani/papers/cache_stampede.pdf) 
- [How to Handle Sudden Bursts of Traffic or "Thundering Herd Problem"?](https://newsletter.scalablethread.com/p/how-to-handle-sudden-bursts-of-traffic?open=false#§queueing-requests)
- [Scaling Instagram Infrastructure - YouTube](https://youtu.be/hnpzNAPiC0E?si=imyLOy4ppNBiCkXb)
- [Cache Made Consistent – Cache Invalidation Might No Longer Be a Hard Thing in Computer Science - YouTube](https://youtu.be/L26RsoslXVg?si=g8P0HiCFKX8a7Zgj)
- [Caches, Promises and Locks - Redis](https://redis.io/blog/caches-promises-locks/)
- [Thundering Herds & Promises. Story of a Service \| by Nick Cooper \| Instagram Engineering](https://instagram-engineering.com/thundering-herds-promises-82191c8af57d)
- [Request collapsing \| Fastly Documentation](https://www.fastly.com/documentation/guides/concepts/edge-state/cache/request-collapsing/)
- [Request and response behavior for custom origins - Amazon CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/RequestAndResponseBehaviorCustomOrigin.html#request-custom-traffic-spikes)
- [How to do distributed locking — Martin Kleppmann’s blog](https://martin.kleppmann.com/2016/02/08/how-to-do-distributed-locking.html)
- [SET \| Docs](https://redis.io/docs/latest/commands/set/)
- [Lease (computer science) - Wikipedia](https://en.wikipedia.org/wiki/Lease_(computer_science))
- [Distributed Locks with Redis \| Docs](https://redis.io/docs/latest/develop/use/patterns/distributed-locks/)
