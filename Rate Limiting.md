---
tags:
  - Distributed-Systems
---

# Token Bucket Algorithm

- The bucket can hold at most b tokens, allowing a maximum burst size of b requests
- A token is added to the bucket at rate r tokens per second. If a token arrives when the bucket is full, it is discarded
- Each request consumes one token. When request arrives, check if there are enough tokens in the bucket:
	- If there are enough tokens, we take one token out for each request, and the request goes through
	- If there are not enough tokens, no tokens are removed from the bucket, and the request is rate limited

# Fixed Window Algorithm

- Timeline is divided into fix-sized time windows with a counter assigned for each window
- Each request increments the counter
- Once the counter reaches the pre-defined limit, new requests are limited until a new time window starts

A drawback of the algorithm is traffic spikes. Bursty traffic at window edges can double the allowed request limit when many requests arrive at the end of one window and the start of the next. For example, with a limit of 5 requests per minute, 5 requests can occur between 00:00 and 01:00, and 5 more between 01:00 and 02:00. For the one minute window from 00:30 to 01:30, 10 requests are processed - twice the allowed limit

# Sliding Window Log Algorithm

- The algorithm records request timestamps within each sliding window
- When a new request comes in, remove all the outdated timestamps - those older than the start of the current sliding window
- Add the new request's timestamp to the sliding window
- If the number of timestamps stored in sliding window is within the allowed limit, the request is accepted. Otherwise, it is rate-limited

This algorithm is the most accurate as it stores each request timestamp and can accurately calculate the rate limits

The major drawback of the algorithm is its high memory consumption because it needs to store a timestamp for each request, event rejected ones

# Sliding Window Counter Algorithm

The sliding window counter algorithm is a hybrid approach that combines the fixed window and sliding window log algorithms

It keeps a request counter for the previous and current fixed windows and uses a sliding window to probabalisitcally calculate the number of requests. The number of requests in the sliding window is calculated using the following formula: 

```
count = countPrev * weight + curCount

countPrev - number of requests in the previous fixed window
weight - overlap percentage of the current window and previous window
curCount - number of requests in the current fixed window
```

# References

- [Diagramming System Design: Rate Limiters](https://www.codesmith.io/blog/diagramming-system-design-rate-limiters#:~:text=The%20sliding%20window%20counter%20requires,not%20always%20be%20the%20case!)
- [System Design Interview - Rate Limiting (local and distributed) - YouTube](https://www.youtube.com/watch?v=FU4WlwfS3G0&t=1743s)
- [7: Design a Rate Limiter \| Systems Design Interview Questions With Ex-Google SWE - YouTube](https://youtu.be/VzW41m4USGs?si=u491wljPnbIyzN3m)
- [Design Rate Limiter System: A Comprehensive Guide](https://systemdesignschool.io/problems/rate-limiter/solution)
- [Scaling your API with rate limiters](https://stripe.com/blog/rate-limiters)
- [How we built rate limiting capable of scaling to millions of domains](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/)
- [Token Bucket Rate Limiting Algorithm](https://www.rdiachenko.com/posts/arch/rate-limiting/token-bucket-algorithm/#)
- [Redis Docs. Pattern: Rate limiter 2](https://redis.io/docs/latest/commands/incr/#pattern-rate-limiter-2)
- [How we scaled the GitHub API with a sharded, replicated rate limiter in Redis - The GitHub Blog](https://github.blog/engineering/how-we-scaled-github-api-sharded-replicated-rate-limiter-redis/)
- [Token bucket - Wikipedia](https://en.wikipedia.org/wiki/Token_bucket)
- [Scaling a High-traffic Rate Limiting Stack with Redis Cluster — brandur.org](https://brandur.org/redis-cluster)
- [Reservation-style rate limiting APIs — brandur.org](https://brandur.org/fragments/reservation-api)
- [Token bucket - Wikipedia](https://en.wikipedia.org/wiki/Token_bucket)
