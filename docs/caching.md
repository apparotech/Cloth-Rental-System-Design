

## 1. What problem are we solving?

Without caching:

```text
User
 ↓
Load Balancer
 ↓
API Server
 ↓
MongoDB
 ↓
Response
```

Imagine 10,000 users request the same popular clothing listing.

MongoDB may repeatedly receive the same query:

```text
Get Clothing A
Get Clothing A
Get Clothing A
Get Clothing A
...
```

This creates unnecessary database load.

With Redis:

```text
User
 ↓
API Server
 ↓
Redis
 ↓
MongoDB
```

If the data is already in Redis, MongoDB isn't queried.

---

# 2. Create `docs/caching.md`

Paste this:

```markdown
# Clothes Rental System - Caching Strategy

## 1. Overview

The Clothes Rental System is expected to be a read-heavy system.

The scale estimation predicts:

- Approximately 10 million registered users
- Approximately 2 million daily active users
- Approximately 3,000 peak API requests per second
- Approximately 580 peak search requests per second

Because many users may repeatedly request the same data,
a caching layer is introduced to reduce repeated database
reads and improve response latency.

Redis is used as the primary caching system.

---

## 2. Why Caching is Required

Without caching, frequently requested data is read directly
from MongoDB.

Example:

Client

↓

API Server

↓

MongoDB

↓

Response

If the same data is requested repeatedly, MongoDB must
process the same read operation multiple times.

This increases:

- Database load
- Query latency
- Infrastructure requirements

Caching allows frequently accessed data to be served
from memory.

---

## 3. Redis

Redis is used as the caching layer between the API servers
and MongoDB.

Basic request flow:

Client

↓

API Server

↓

Redis

↓

MongoDB

Redis provides low-latency access to frequently requested
data.

Because Redis is shared between API servers, all API
instances can access the same cached data.

---

## 4. Cache-Aside Pattern

The system uses the Cache-Aside pattern for most read
operations.

Flow:

Client

↓

API Server

↓

Check Redis

↓

Cache Hit → Return data

Cache Miss → Query MongoDB

↓

Store result in Redis

↓

Return response

The application is responsible for reading from and writing
to the cache.

---

## 5. Cache Hit

A cache hit occurs when the requested data already exists
in Redis.

Example:

User requests clothing details.

API Server

↓

Redis

↓

Data found

↓

Return response

MongoDB is not queried.

---

## 6. Cache Miss

A cache miss occurs when the requested data is not available
in Redis.

Flow:

API Server

↓

Redis

↓

Cache Miss

↓

MongoDB

↓

Fetch data

↓

Store data in Redis

↓

Return response

The next request can then be served directly from Redis.

---

## 7. Data Suitable for Caching

Not all data should be cached.

The following data is suitable for caching:

### Clothing Details

Frequently viewed clothing listings can be cached.

Example key:

`clothing:{clothingId}`

---

### Popular Clothing Listings

Frequently accessed listing results can be cached.

Example:

`clothing:list:{hash}`

The hash represents the query parameters.

---

### Subscription Plans

Subscription plans change relatively infrequently and can
be cached for longer periods.

Example:

`subscription:plans`

---

### Frequently Accessed Public Data

Examples:

- Clothing categories
- Popular listings
- Public clothing details
- Public review summaries

---

## 8. Data That Should Not Rely on Cache

Highly consistent data should continue to use MongoDB as
the source of truth.

Examples:

- Booking availability
- Booking creation
- Booking status
- Payment status
- Subscription status during critical operations

Caching must not be used as the final source of truth for
booking availability.

For example:

Redis should not decide whether a clothing item can be
booked.

The booking system must perform the authoritative
availability check against the database.

---

## 9. Cache Key Design

Cache keys should be predictable and uniquely identify
the cached data.

Examples:

`clothing:{clothingId}`

`user:{userId}`

`subscription:plans`

`clothing:list:{queryHash}`

Using structured keys makes cache management easier.

---

## 10. TTL

Cached data should have a Time-To-Live (TTL).

After the TTL expires, Redis automatically removes the
cached value.

Example:

Clothing details

TTL = 5 minutes

Subscription plans

TTL = 1 hour

The exact TTL values are configuration decisions and may
change based on production traffic and update frequency.

---

## 11. Cache Invalidation

When cached data changes, stale cache data must be handled.

Example:

User updates clothing price.

Before update:

MongoDB

`price = ₹800`

Redis

`price = ₹800`

User changes price:

`₹800 → ₹1000`

The application updates MongoDB and invalidates the
corresponding Redis key.

Example:

`DEL clothing:{clothingId}`

The next request results in a cache miss and loads the
updated data from MongoDB.

---

## 12. Read Flow

Typical read flow:

Client

↓

API Server

↓

Redis

↓

┌───────────────┐
│ Cache exists? │
└───────┬───────┘
        │
    ┌───┴────┐
    ↓        ↓
   YES       NO
    ↓        ↓
 Return   MongoDB
            ↓
        Store in Redis
            ↓
         Return
```

---

## 13. Write Flow

For data that is cached, updates should keep the cache
consistent with the database.

Example:

Client

↓

API Server

↓

Update MongoDB

↓

Invalidate Redis Cache

The next read will fetch the latest data from MongoDB
and repopulate Redis.

This approach avoids returning outdated cached data after
a successful update.

---

## 14. Cache Consistency

Redis is treated as a performance optimization and not as
the primary source of truth.

MongoDB remains the source of truth for persistent
application data.

If Redis becomes unavailable, the application should be
able to continue operating by reading from MongoDB, although
with increased latency and database load.

---

## 15. Cache Failure

If Redis is temporarily unavailable:

Client

↓

API Server

↓

Redis unavailable

↓

MongoDB

↓

Response

The API should have a fallback path where appropriate.

Redis failure should not cause permanent data loss because
the persistent data is stored in MongoDB.

---

## 16. Cache Stampede

A cache stampede can occur when a popular cache entry expires
and many users request the same data at approximately the
same time.

Example:

Popular clothing cache expires.

↓

1,000 requests arrive.

↓

All requests miss Redis.

↓

1,000 requests query MongoDB.

This can create a sudden database load spike.

Possible mitigation techniques include:

* Request coalescing
* Distributed locking
* Cache prewarming
* Randomized TTL
* Background refresh

The exact technique can be selected based on traffic
patterns and production requirements.

---

## 17. Cache Eviction

Redis has limited memory.

When memory becomes constrained, Redis may evict cached
entries according to the configured eviction policy.

Because Redis contains derived/cache data rather than the
primary persistent data, evicted entries can be recreated
from MongoDB.

---

## 18. Horizontal Scaling of Redis

As traffic grows, Redis can be scaled independently from
the API servers.

Possible approaches include:

* Redis replication
* Redis Sentinel
* Redis Cluster

Redis Cluster can distribute keys across multiple Redis
nodes for larger datasets and higher throughput.

The exact deployment model depends on the required
availability, dataset size, and traffic.

---

## 19. Important Design Principle

Caching should be applied selectively.

The system should not cache every database query.

The main goals are:

* Reduce repeated database reads
* Improve read latency
* Reduce database load
* Improve scalability

Critical operations requiring strong consistency should
continue to use the authoritative database state.

---

## 20. Summary

The Clothes Rental System uses Redis as a caching layer
between API servers and MongoDB.

The main strategy is:

Client

↓

API Server

↓

Redis

↓

MongoDB on cache miss

The cache is used primarily for read-heavy and frequently
accessed data.

MongoDB remains the source of truth for persistent data,
while Redis improves performance and reduces database load.

