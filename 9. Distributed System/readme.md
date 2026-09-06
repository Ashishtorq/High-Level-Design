# Introduction to Distributed Systems

## Summary

A distributed system is a group of independent computers that communicate and coordinate over a network to behave like one system from the user's perspective. Instead of relying on a single machine, the workload and data are spread across multiple nodes.

This architecture helps applications scale to support more users, remain operational when individual machines fail, and reduce latency by serving users from geographically closer locations. The main challenge is coordination: because each node operates independently and network communication is imperfect, the nodes must agree on shared state and handle failures without corrupting data.

The CAP theorem describes an important trade-off in distributed systems. When a network partition prevents nodes from communicating reliably, a system cannot guarantee both complete consistency and full availability at the same time.

## 1. What Is a Distributed System?

A distributed system consists of multiple independent computers, commonly called **nodes**, that work together through a network.

Each node has its own:

- Processor
- Memory
- Local state
- Operating system
- Possibility of failing independently

Although several computers are involved internally, the system should appear to the user as a single application or service.

```text
Users -> Distributed system -> Multiple cooperating nodes
```

### Simple Example

Suppose an application becomes too busy for one server. Instead of replacing it with a much larger machine, the application can run on several servers. Incoming requests are distributed among them, allowing the servers to share the workload.

## 2. Why Use Distributed Systems?

A single machine has limited processing power, memory, storage, and network capacity. It can also become a single point of failure.

Distributed systems address these limits by adding more machines and coordinating their work. Their main benefits include:

- Scalability
- Fault tolerance
- Lower latency
- Higher throughput
- Better geographic reach

These benefits come with additional complexity, especially around communication, coordination, consistency, and failure handling.

## 3. Scalability

Scalability is the ability of a system to continue handling increasing traffic or workload.

### Vertical Scaling

Vertical scaling means increasing the resources of one machine, such as adding more CPU, memory, or storage.

**Advantages:**

- Simple architecture
- Often requires fewer application changes

**Limitations:**

- Hardware has a maximum capacity
- Powerful machines can be expensive
- The machine may remain a single point of failure

### Horizontal Scaling

Horizontal scaling means adding more machines or server instances.

```text
                 -> Server A
User requests -> Load balancer -> Server B
                 -> Server C
```

A load balancer can route incoming traffic across the available servers.

**Advantages:**

- Capacity can grow by adding nodes
- Traffic is distributed across machines
- Individual servers can be replaced or removed
- Supports very large workloads

**Challenges:**

- Shared data must remain consistent
- Requests may reach different servers
- Nodes must communicate and coordinate
- Deployment and monitoring become more complex

Distributed systems commonly use horizontal scaling because it allows an application to grow beyond the limits of one machine.

## 4. Fault Tolerance

Fault tolerance is the ability of a system to continue operating when one or more components fail.

In a single-server architecture, a server crash may make the entire application unavailable. This server is a **single point of failure**.

In a distributed architecture, multiple nodes can provide the same service. If one node fails, traffic can be routed to healthy nodes.

```text
Server A: Available
Server B: Failed
Server C: Available

New requests -> Server A or Server C
```

### Techniques That Support Fault Tolerance

- Running multiple application instances
- Replicating important data
- Performing health checks
- Automatically rerouting traffic
- Retrying failed operations carefully
- Replacing unhealthy nodes
- Avoiding dependence on a single service or database instance

### Important Distinction

Having multiple servers does not automatically guarantee fault tolerance. The system must detect failures, route around them, and ensure that redundant components do not share the same failure risk.

For example, several servers in one data center may all become unavailable during a regional outage.

## 5. Low Latency Through Geographic Distribution

Latency is the time required for a request to travel through the system and receive a response.

Physical distance affects network latency. If an application is hosted only in one region, users located far away may experience slower responses.

A geographically distributed system can place application servers or cached data closer to users.

```text
Users in Asia   -> Asia region
Users in Europe -> Europe region
Users in America -> America region
```

### Benefits

- Shorter network distance
- Faster response times
- Better experience for global users
- Continued service if one region becomes unavailable

### Challenges

- Data must be replicated between regions
- Replication takes time
- Different regions may temporarily hold different values
- Cross-region coordination increases latency
- Data residency and compliance requirements may restrict where data is stored

Geographic distribution reduces user-facing latency, but synchronizing data across distant regions introduces consistency trade-offs.

## 6. Agreement and Coordination

Independent nodes often need to agree on the state of the system before completing an operation.

### Inventory Example

Suppose an online store has one item remaining and two users attempt to purchase it through different application servers.

Without coordination:

1. Server A reads that one item is available.
2. Server B also reads that one item is available.
3. Both servers accept a purchase.
4. The system sells more inventory than it owns.

The servers therefore need a coordination mechanism that ensures only one purchase succeeds.

### Common Coordination Tools

- Atomic database transactions
- Conditional writes
- Unique constraints
- Distributed locks
- Consensus protocols
- A single leader for ordered writes
- Version numbers or optimistic concurrency control

### Why Agreement Is Difficult

Nodes communicate over a network, and networks are not perfectly reliable. Messages may be:

- Delayed
- Lost
- Duplicated
- Delivered out of order
- Blocked by a network partition

A slow response also creates ambiguity. A node may be slow, disconnected, or completely unavailable, and another node cannot always immediately determine which condition is true.

## 7. The CAP Theorem

The CAP theorem describes three properties:

### Consistency

Every successful read receives the latest successful write, or the operation fails. All clients observe a compatible view of the data.

### Availability

Every request to a non-failing node receives a response, although the response may not contain the latest data.

### Partition Tolerance

The system continues operating despite a network partition that prevents some nodes from communicating with others.

### The Actual Trade-Off

CAP is often described as “choose any two of the three,” but that wording can be misleading. In a real distributed system, network partitions can occur and must be handled.

When a partition occurs, the system must choose between:

- **Consistency:** Reject or delay some operations until the nodes can coordinate.
- **Availability:** Continue accepting operations, with the risk that different nodes temporarily return different data.

| Choice During a Partition | Behavior | Trade-Off |
|---|---|---|
| Prefer consistency | Some requests may fail or wait | Reduced availability |
| Prefer availability | Nodes continue responding | Temporary inconsistency is possible |

The appropriate choice depends on the feature. A banking transaction may prioritize consistency, while a social media feed may tolerate briefly stale data to remain available.

## 8. Core Challenges

Distributed systems must deal with problems that are less significant in a single-machine application:

- Partial failures, where some nodes fail while others continue working
- Unreliable or delayed network communication
- Concurrent operations on shared data
- Data replication and synchronization
- Ordering events across different machines
- Duplicate requests and retries
- Detecting whether a node is slow or unavailable
- Operating and monitoring many components

Because failures are unavoidable, distributed systems should be designed with the expectation that machines, processes, and networks can fail.

## 9. Key Design Trade-Offs

| Goal | Benefit | Cost or Challenge |
|---|---|---|
| Add more nodes | Greater capacity | More coordination complexity |
| Replicate services | Better fault tolerance | More infrastructure to operate |
| Replicate data | Better availability and read performance | Risk of stale or conflicting copies |
| Use multiple regions | Lower user latency | Higher synchronization latency |
| Require immediate agreement | Stronger data integrity | Slower operations or reduced availability |
| Allow eventual consistency | Faster and more available operations | Clients may temporarily see stale data |

## 10. Practical Takeaways

- A distributed system uses multiple independent computers to provide one logical service.
- Horizontal scaling helps handle increasing traffic by adding server instances.
- Redundancy removes single points of failure only when failures can be detected and traffic can be rerouted.
- Geographic distribution can reduce latency for global users.
- Shared state requires coordination to prevent conflicts such as overselling inventory.
- Network calls can fail or be delayed, so failure handling is part of the normal design.
- CAP describes the consistency-versus-availability decision made during a network partition.
- Distributed systems improve scale and resilience, but they introduce operational and data-consistency complexity.

## Quick Revision

- **Distributed system:** Independent nodes working together as one logical system.
- **Node:** An individual computer or process participating in the system.
- **Scalability:** The ability to handle increasing workload.
- **Horizontal scaling:** Adding more machines or application instances.
- **Fault tolerance:** Continuing to operate despite component failures.
- **Single point of failure:** One component whose failure can stop the entire system.
- **Latency:** The time taken to process and respond to a request.
- **Agreement:** Coordination between nodes about shared state or operation order.
- **Consistency:** Reads reflect the latest successful write.
- **Availability:** Requests to non-failing nodes receive responses.
- **Partition tolerance:** The system continues to handle network communication failures.
- **CAP trade-off:** During a partition, the system must sacrifice either consistency or availability.
