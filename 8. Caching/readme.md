# Caching and Distributed Caching

## Summary

Caching stores frequently accessed data in fast, temporary storage so applications do not need to repeatedly query a slower database or external service. This reduces latency, lowers database load, and improves scalability.

A local cache is extremely fast because it lives inside an application server, but it becomes difficult to keep consistent when multiple servers are running. A distributed cache, such as Redis, gives all application servers access to the same cached data. This improves consistency across the system, although each cache request requires a small network call.

Good cache design also requires choosing how data is removed when space is limited and how updates are synchronized with the database. Eviction policies such as FIFO, LRU, and LFU decide what to remove. Write-through and write-back policies balance consistency, performance, and durability in different ways.

## 1. What Is Caching?

Caching is the practice of storing frequently requested data in a fast, temporary storage layer, usually in memory.

Without caching:

1. The application receives a request.
2. It queries the database or another remote service.
3. It returns the result to the client.

With caching:

1. The application first checks the cache.
2. If the data exists, it returns the cached value.
3. If the data does not exist, it fetches it from the database, stores it in the cache, and returns it.

### Benefits

- Lower response time
- Fewer database queries
- Reduced database and network load
- Better throughput and scalability
- Improved user experience

### Important Limitation

Cached data is a copy of the original data. If the database changes but the cache does not, users may receive stale data. Cache invalidation and update policies are therefore essential.

## 2. Cache Hit and Cache Miss

### Cache Hit

A cache hit occurs when the requested data is already available in the cache.

```text
Client -> Application -> Cache -> Cached result
```

This is the ideal path because it avoids the slower database query.

### Cache Miss

A cache miss occurs when the requested data is not present in the cache.

```text
Client -> Application -> Cache miss -> Database
                              |
                              -> Store result in cache
```

The first request is slower, but later requests can use the cached value.

### Useful Metric: Cache Hit Ratio

The cache hit ratio measures how often requests are served from the cache.

```text
Cache hit ratio = Cache hits / Total cache lookups
```

A higher hit ratio generally means the cache is reducing more database work, but it should be evaluated together with data freshness, memory use, and latency.

## 3. Local Cache

A local cache stores data directly in the memory of an application server.

### Advantages

- Very low latency
- No additional network call
- Simple for a single-server application
- Reduces repeated work within the same server

### Disadvantages

- Each server maintains a separate copy of the data
- Different servers may return different versions of the same value
- Cache contents disappear when the server restarts
- Memory capacity is limited to the individual server
- Invalidation must be coordinated across all servers

### Example

Suppose two application servers cache the same user profile. If the profile is updated through Server A, Server B may continue returning its older cached copy. This creates inconsistency.

### Best Fit

Local caching is useful for:

- Single-server applications
- Data that rarely changes
- Server-specific data
- Small, frequently reused values
- A first-level cache placed in front of a distributed cache

## 4. Distributed Cache

A distributed cache is a shared cache service accessed by multiple application servers. Redis and Memcached are common examples.

```text
Application Server A --\
Application Server B ----> Shared Distributed Cache -> Database
Application Server C --/
```

### Advantages

- All application servers use a shared cached value
- Easier consistency across a scaled application
- Cache capacity can be managed independently of application servers
- Supports horizontal application scaling
- Centralizes expiration and eviction behavior

### Disadvantages

- Adds network latency to cache operations
- Can become a bottleneck or failure point if poorly designed
- Requires separate deployment, monitoring, and scaling
- Network failures can make the cache temporarily unavailable

### Best Fit

Distributed caching is useful when:

- The application runs on multiple servers
- Cached data must be shared across instances
- The system handles high read traffic
- Reducing database load is important
- Cache capacity must scale independently

## 5. Local Cache vs. Distributed Cache

| Factor | Local Cache | Distributed Cache |
|---|---|---|
| Location | Inside an application server | Separate shared service |
| Speed | Extremely fast | Fast, but requires a network call |
| Data sharing | Limited to one server | Shared by all application servers |
| Consistency | Harder across multiple servers | Easier across multiple servers |
| Capacity | Limited by server memory | Can scale independently |
| Failure behavior | Lost when the server restarts | Depends on the cache cluster and persistence setup |
| Operational complexity | Low | Higher |

## 6. Cache Eviction Policies

Eviction is the removal of cached entries when the cache reaches its memory limit. The eviction policy determines which entries are removed.

### FIFO: First In, First Out

FIFO removes the entry that was added earliest.

**Advantages:**

- Simple to implement
- Predictable behavior
- Low tracking overhead

**Disadvantages:**

- Ignores how recently or frequently an entry is used
- May remove popular data simply because it is old

### LRU: Least Recently Used

LRU removes the entry that has not been accessed for the longest time.

**Advantages:**

- Works well when recently accessed data is likely to be accessed again
- Commonly effective for real-world workloads

**Disadvantages:**

- Requires tracking recent access
- A large one-time scan can push useful older data out of the cache

### LFU: Least Frequently Used

LFU removes the entry with the lowest access frequency.

**Advantages:**

- Retains consistently popular data
- Useful when access popularity remains stable over time

**Disadvantages:**

- Requires frequency tracking
- Previously popular data may remain cached after it stops being useful unless frequency scores decay

### Comparison

| Policy | Removes | Best When | Main Risk |
|---|---|---|---|
| FIFO | Oldest inserted entry | Simplicity matters | May remove frequently used data |
| LRU | Least recently accessed entry | Recent use predicts future use | One-time scans can pollute the cache |
| LFU | Least frequently accessed entry | Long-term popularity matters | Old popularity can dominate |

## 7. Cache Write Policies

Write policies define how changes are propagated between the cache and the database.

### Write-Through

In write-through caching, a write updates the cache and the database as part of the same request path. The write is considered successful only after the required updates complete.

```text
Application -> Cache -> Database
```

**Advantages:**

- Stronger consistency between the cache and database
- Recently written data is immediately available in the cache
- Simpler reads because the cache stays current

**Disadvantages:**

- Higher write latency
- Every write reaches the database
- The cache may store data that is rarely read

### Write-Back

In write-back caching, the application updates the cache first. The database is updated asynchronously later.

```text
Application -> Cache -> Asynchronous database update
```

**Advantages:**

- Very fast writes
- Multiple updates can be batched before reaching the database
- Reduces immediate database write load

**Disadvantages:**

- The cache and database may temporarily contain different values
- Data can be lost if the cache fails before pending writes reach the database
- Requires queues, retries, ordering, and failure handling
- More complex to operate correctly

### Comparison

| Factor | Write-Through | Write-Back |
|---|---|---|
| Write latency | Higher | Lower |
| Consistency | Stronger | Eventual |
| Database update | Immediate | Asynchronous |
| Data-loss risk | Lower | Higher if pending writes are not durable |
| Complexity | Moderate | High |
| Best fit | Consistency-sensitive systems | Write-heavy systems that can tolerate delayed persistence |

## 8. Key Design Trade-Offs

Caching improves speed, but every design must balance:

- **Latency:** Local caches are fastest, while distributed caches require a network call.
- **Consistency:** Cached copies can become stale when the source data changes.
- **Availability:** Applications should define what happens when the cache is unavailable.
- **Capacity:** Limited memory makes eviction necessary.
- **Durability:** A cache should usually not be treated as the only permanent copy of important data.
- **Complexity:** Distributed caches and asynchronous writes require monitoring and failure handling.

## 9. Practical Takeaways

- Use caching for frequently read or expensive-to-compute data.
- Treat the database as the source of truth unless the architecture explicitly provides durable write-back behavior.
- Use local caching when the data is server-specific or minor inconsistency is acceptable.
- Use distributed caching when multiple application servers need shared cached data.
- Choose an eviction policy based on the application's access pattern.
- Use write-through when consistency matters more than write speed.
- Use write-back only when the system can tolerate eventual consistency and safely handle cache failures.
- Define expiration and invalidation rules so cached values do not remain stale indefinitely.
- Monitor hit ratio, miss ratio, memory usage, eviction count, error rate, and cache latency.

## Quick Revision

- **Cache:** Fast temporary storage for frequently accessed data.
- **Cache hit:** Requested data is found in the cache.
- **Cache miss:** Data must be fetched from the source and can then be cached.
- **Local cache:** Fast, server-specific, and harder to keep consistent at scale.
- **Distributed cache:** Shared across servers, more consistent, but requires a network call.
- **FIFO:** Removes the oldest inserted entry.
- **LRU:** Removes the least recently accessed entry.
- **LFU:** Removes the least frequently accessed entry.
- **Write-through:** Updates the cache and database in the request path.
- **Write-back:** Updates the cache first and persists to the database asynchronously.
