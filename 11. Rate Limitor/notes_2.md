# Rate Limiting Algorithms: Part 2

## Summary

Rate limiting controls how many requests a client can send within a period of time. It protects services from traffic spikes, abuse, accidental retry storms, and resource exhaustion.

This note covers three window-based algorithms:

- *Fixed Window Counter:* Simple and memory-efficient, but vulnerable to bursts at window boundaries.
- *Sliding Window Log:* Precise, but stores a timestamp for every accepted request and can consume significant memory.
- *Sliding Window Counter:* Approximates a true sliding window using counters from the current and previous fixed windows, balancing accuracy with memory efficiency.

There is no universally best algorithm. The right choice depends on the required precision, expected traffic, storage cost, and tolerance for short bursts.

## 1. What Is Rate Limiting?

A rate limiter decides whether a request should be allowed based on a rule such as:

text
Allow at most 100 requests per user per minute.


For every incoming request, the limiter returns one of two decisions:

- *Allow:* Process the request normally.
- *Reject or delay:* The client has exceeded the configured limit.

Rejected HTTP requests commonly receive status code 429 Too Many Requests.

mermaid
flowchart LR
    C[Client] --> R[Rate limiter]
    R -->|Within limit| A[Application]
    R -->|Limit exceeded| X["429 Too Many Requests"]


### Common Rate-Limit Keys

A limit can be tracked by:

- User ID
- API key
- IP address
- Account or organization
- Endpoint
- A combination such as user_id + endpoint

## 2. Fixed Window Counter

The fixed window counter divides time into non-overlapping intervals. Each key has one counter for the active interval.

For a limit of 100 requests per minute:

- One window might cover elapsed seconds 0 through 59.
- The next window covers elapsed seconds 60 through 119.
- The counter resets when a new window begins.

### How It Works

1. Determine the fixed window containing the current time.
2. Read the counter for the client and that window.
3. If the counter is below the limit, increment it and allow the request.
4. Otherwise, reject the request.
5. Start a fresh counter when the next window begins.

mermaid
flowchart TD
    Q[Request arrives] --> W[Find current fixed window]
    W --> C{Counter below limit?}
    C -->|Yes| I[Increment and allow]
    C -->|No| R[Reject]


### Simplified Pseudocode

text
window_id = floor(current_time / window_size)
key = client_id + window_id
count = increment(key)

if count <= limit:
    allow request
else:
    reject request


The check and increment should be atomic. Otherwise, concurrent requests can read the same old count and collectively exceed the limit.

### The Boundary Burst Problem

Suppose the limit is 100 requests per minute:

- A client sends 100 requests near the end of one minute.
- The counter resets at the next minute boundary.
- The client immediately sends another 100 requests.

The limiter accepts 200 requests in a very short real-time interval, even though each fixed window contains only 100.

mermaid
timeline
    title Fixed-window boundary burst
    Previous window : 100 requests near its end
    Boundary : Counter resets
    Current window : 100 requests near its start


The system therefore permits a burst approaching twice the configured limit around a boundary.

### Advantages

- Simple to understand and implement
- Stores only one counter per active key and window
- Fast constant-time checks in typical implementations
- Expired counters can be removed automatically

### Disadvantages

- Allows large bursts around window boundaries
- Fairness depends on arbitrary clock-aligned intervals
- Does not represent the true request count over the immediately preceding duration

### Best Fit

Use a fixed window counter when:

- Simplicity is the priority
- Short bursts are acceptable
- The limit protects quotas more than instantaneous capacity
- Approximate enforcement is sufficient

## 3. Sliding Window Log

The sliding window log records the timestamp of every accepted request. For each new request, the limiter examines the interval immediately preceding the current time.

For a one-minute window evaluated at elapsed second 90, the relevant interval begins at second 30. Requests older than that are expired.

### How It Works

1. Load the timestamp log for the client.
2. Remove timestamps older than the sliding-window cutoff.
3. Count the remaining timestamps.
4. If the count is below the limit, append the current timestamp and allow the request.
5. Otherwise, reject the request.

mermaid
flowchart TD
    Q[Request arrives] --> P[Remove expired timestamps]
    P --> C{Remaining count below limit?}
    C -->|Yes| A[Append timestamp and allow]
    C -->|No| R[Reject]


### Simplified Pseudocode

text
cutoff = current_time - window_size
remove timestamps older than cutoff

if log.size < limit:
    append current_time
    allow request
else:
    reject request


### Example

Assume a limit of three requests in any ten-second interval.

text
Stored request times: 02s, 05s, 09s
New request time:     12s
Cutoff time:           2s


Depending on whether the lower boundary is inclusive, the request at 02s is removed or retained. The implementation must define boundary semantics consistently.

After pruning expired entries, the limiter checks the exact number of relevant requests. This avoids the artificial reset used by a fixed window.

### Advantages

- Accurately enforces the limit over any rolling interval
- Eliminates the fixed-window boundary burst
- Makes recent request history directly observable

### Disadvantages

- Stores one timestamp for every retained request
- Memory usage grows with traffic volume and the configured limit
- Requires timestamp insertion, removal, and counting
- Distributed implementations need atomic pruning, counting, and insertion

### Memory Behavior

For a limit of L requests per window, an implementation may retain approximately O(L) timestamps for each active key. This becomes expensive when there are many users or large limits.

### Best Fit

Use a sliding window log when:

- Exact enforcement is required
- Limits are relatively small
- Memory cost is acceptable
- Boundary bursts must be prevented

## 4. Sliding Window Counter

The sliding window counter is a memory-efficient approximation of a sliding window. It usually keeps only:

- The count from the previous fixed window
- The count from the current fixed window
- The current position within the window

Instead of storing every timestamp, it estimates how much of the previous window overlaps the current sliding interval.

### Estimation Formula

Let:

- previous_count be the previous fixed-window count.
- current_count be the current fixed-window count.
- elapsed be the time elapsed in the current window.
- window_size be the full window duration.

The estimated rolling count is:

text
previous_weight = (window_size - elapsed) / window_size

estimated_count =
    current_count + (previous_count × previous_weight)


The previous count receives less weight as the current window progresses.

### Worked Example

Assume:

- Limit: 100 requests per minute
- Previous-window count: 80
- Current-window count: 30
- Time elapsed in the current minute: 15 seconds

The previous minute overlaps the current sliding interval by 45 of 60 seconds:

text
previous_weight = (60 - 15) / 60
                = 0.75

estimated_count = 30 + (80 × 0.75)
                = 90


The estimated count is 90. If the algorithm evaluates the new request before incrementing, it can allow the request because the new estimate would remain within the limit.

### How It Works

1. Identify the current fixed window.
2. Retrieve the current and previous window counters.
3. Calculate how much of the previous window overlaps the rolling interval.
4. Compute the weighted estimate.
5. Allow and increment if the resulting count stays within the limit.
6. Otherwise, reject the request.

mermaid
flowchart TD
    Q[Request arrives] --> D[Read current and previous counters]
    D --> W[Weight previous count by overlap]
    W --> E[Calculate estimated rolling count]
    E --> C{New count within limit?}
    C -->|Yes| A[Increment current counter and allow]
    C -->|No| R[Reject]


### Why It Is an Approximation

The algorithm assumes requests in the previous window were distributed relatively evenly. It knows the total count but not the exact timestamps.

If the previous requests were heavily concentrated in the overlapping or non-overlapping part of that window, the estimate may differ from the true rolling count.

### Advantages

- Uses much less memory than a sliding log
- Reduces the fixed-window boundary burst
- Requires only a small number of counters per key
- Provides smoother enforcement than a fixed window

### Disadvantages

- Does not calculate the exact rolling count
- Accuracy depends on how requests were distributed in the previous window
- Slightly more complex than a fixed window counter
- Concurrent distributed updates still require atomic operations

### Best Fit

Use a sliding window counter when:

- Fixed-window bursts are unacceptable
- A close approximation is sufficient
- Sliding-log memory usage is too expensive
- The system needs a practical balance of accuracy, speed, and storage cost

## 5. Algorithm Comparison

| Factor | Fixed Window Counter | Sliding Window Log | Sliding Window Counter |
|---|---|---|---|
| Accuracy | Low near boundaries | Exact | Approximate but usually close |
| Memory per active key | Very low | High | Low |
| Implementation complexity | Low | Moderate to high | Moderate |
| Boundary burst protection | Poor | Strong | Stronger than fixed window |
| Stored data | Current counter | Timestamp per request | Current and previous counters |
| Typical operation cost | Constant time | Depends on log operations | Constant time |
| Best use | Simple quotas | Strict enforcement | General-purpose rate limiting |

## 6. Visual Comparison

Consider a limit of five requests per ten seconds.

text
Time:     0----5----10---15---20
Requests:         ***** *****
                  end   start


- *Fixed window:* May accept both groups because the counter resets at time 10.
- *Sliding log:* Counts the exact timestamps in the previous ten seconds and rejects requests beyond five.
- *Sliding counter:* Uses a weighted portion of the previous counter and usually rejects most of the boundary burst, but the result is estimated.
 
## 7. Distributed-System Considerations

The algorithm alone does not make a distributed rate limiter correct. When several application servers process requests for the same client, they need shared or coordinated state.

### Shared State

A distributed data store such as Redis can hold counters or timestamp logs that all application instances access.

### Atomicity

The following steps must behave atomically:

- Read or prune the current state
- Check the limit
- Record the accepted request
- Set or preserve expiration

Without atomicity, concurrent requests may all observe an available slot and exceed the limit.

### Expiration

Counter keys and logs should expire after they are no longer relevant. Otherwise, inactive clients leave unused state in storage indefinitely.

### Clock Behavior

Timestamp-based algorithms depend on time. Servers should use a consistent time source, and designs should account for clock skew or time adjustments.

### Failure Policy

If the rate-limit store becomes unavailable, the service must choose a policy:

- *Fail open:* Allow requests, preserving availability but reducing protection.
- *Fail closed:* Reject requests, protecting the backend but reducing availability.

The correct choice depends on the endpoint. A public content endpoint and a sensitive authentication endpoint may use different policies.

## 8. Practical Design Guidance

- Start with the simplest algorithm that satisfies the service requirements.
- Use a fixed window for inexpensive quotas where short boundary bursts are safe.
- Use a sliding log when exact rolling-window enforcement justifies the memory cost.
- Use a sliding counter for a strong balance between accuracy and efficiency.
- Define whether the request that reaches the limit is allowed or rejected.
- Define interval-boundary semantics explicitly.
- Make distributed updates atomic.
- Return useful metadata such as the remaining quota and retry time when appropriate.
- Monitor rejection rates, hot keys, storage latency, and limiter failures.
- Apply separate limits to different users, operations, or resource costs when one global rule would be unfair.

## 9. Common Interview Questions

### Why can a fixed window allow twice the configured rate?

Because the counter resets at a fixed boundary. A client can use the entire allowance immediately before the reset and the entire next allowance immediately after it.

### Why is a sliding window log accurate?

It stores individual request timestamps and counts only requests that fall inside the exact rolling interval.

### Why is a sliding window log memory-intensive?

It retains a separate timestamp for every relevant request rather than storing only aggregate counters.

### How does the sliding window counter save memory?

It stores aggregate counts for two windows and weights the previous count according to its overlap with the current rolling interval.

### Is the sliding window counter exact?

No. It estimates the previous window's contribution because it does not know when individual requests occurred inside that window.

### Which algorithm should be used by default?

The sliding window counter is often a practical general-purpose choice, but requirements decide the answer. Use the fixed window when simplicity is enough and the sliding log when exact enforcement is necessary.

## Quick Revision

- *Fixed window counter:* One counter per fixed interval; simple but vulnerable at boundaries.
- *Sliding window log:* One timestamp per request; accurate but memory-intensive.
- *Sliding window counter:* Weighted current and previous counters; efficient but approximate.
- *Boundary burst:* Traffic concentrated around a counter reset can exceed the intended rolling rate.
- *Atomic update:* The limit check and state update must not race with other requests.
- *Fail open:* Allow traffic if the limiter fails.
- *Fail closed:* Reject traffic if the limiter fails.