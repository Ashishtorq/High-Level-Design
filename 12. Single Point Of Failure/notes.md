# Single Points of Failure (SPOF) in Distributed Systems

## What is a Single Point of Failure?

A **Single Point of Failure (SPOF)** is a component whose failure can make the entire system, or a critical function, unavailable.

Examples include a single application server, database, load balancer, or shared network connection. Failure can cause downtime, failed requests, potential data loss, and a poor user experience.

## Strategies to Reduce SPOFs

### 1. Redundancy

Run multiple instances of critical components so that healthy instances can take over when one fails.

- **Application servers:** Use horizontal scaling to run the application on multiple servers.
- **Load balancers:** Use a highly available load-balancing setup; one load balancer can itself become an SPOF.
- **Health checks:** Detect unhealthy instances and stop routing requests to them.
- **Spare capacity:** Ensure the remaining instances can handle the load after a failure.

**Example:** If one of three application servers fails, traffic is routed to the other two.

### 2. Data Replication and Database Failover

Maintain copies of data on multiple database nodes. A common approach is **primary-replica replication**, also called master-slave replication.

- The primary typically handles writes.
- Replicas receive copies of the primary's data and may serve reads.
- If the primary fails, a suitable replica can be promoted to become the new primary.

**Important:** Replication alone does not guarantee availability. The system also needs failure detection, safe promotion, and a way to direct clients to the new primary.

With asynchronous replication, replicas can lag behind the primary. Promoting a replica may therefore lose recent writes that had not yet replicated. Preventing multiple nodes from acting as the writable primary is also essential to avoid conflicting writes.

### 3. Geographic Distribution

Spread components across independent locations to reduce the impact of shared infrastructure failures.

| Deployment scope | Failure it helps tolerate |
| --- | --- |
| Multiple machines | Failure of an individual server |
| Multiple availability zones | An outage affecting one zone's infrastructure |
| Multiple geographic regions | A broader regional outage |

**Example:** Multiple servers in one data center may all become unavailable during the same power or network outage. Spreading them across independent locations reduces this risk.

Multi-region designs add cost and complexity, especially around data consistency, replication latency, and traffic failover. Choose the scope based on the system's availability requirements.

### 4. Graceful Degradation

Keep essential features working when a non-critical component fails.

**Example:** If a recommendation engine goes down, customers should still be able to browse products and place orders. The application can hide recommendations or show a default list.

Use timeouts and fallback behavior so that an optional dependency does not block the entire request.

## Additional Practical Notes

### Redundancy Must Cover Shared Dependencies

Two application servers are not fully independent if both rely on the same database, physical host, storage device, or network connection.

Trace the complete request path and ask: **If this component fails, can the critical user operation still complete?**

### Replication Is Not a Backup

Replication can copy accidental deletions or corrupted data to other nodes. Keep separate backups and test that they can be restored.

### Prevent Cascading Failures

A slow or failing dependency can consume connections, threads, or memory until other services fail too.

- Set timeouts for remote calls.
- Limit retries and use backoff with jitter to avoid overwhelming a struggling service.
- Use circuit breakers to temporarily stop calls to a repeatedly failing dependency.
- Isolate resources so that one failing feature cannot exhaust resources needed by others.

### Test Recovery

Monitor service health and test failover under realistic load. Having a standby instance is useful only if it can actually take over.

Define two recovery targets:

- **Recovery Time Objective (RTO):** The target maximum time to restore service after an outage.
- **Recovery Point Objective (RPO):** The target maximum amount of data loss, measured in time.

## Key Takeaway

The goal is to reduce critical failure points and limit the impact of failures. Resilience comes from **redundancy, replication, independent failure domains, tested failover, and graceful degradation** working together.

Adding more servers alone is not enough; the system must detect failures and recover safely.
