# Scaling in System Design

## Overview

Scaling means increasing a system's capacity when it can no longer handle growing user traffic or workloads efficiently.

There are two primary approaches: **vertical scaling** and **horizontal scaling**.

## Vertical Scaling

Vertical scaling, also called **scaling up**, increases the capacity of an existing server.

Examples include:

- Adding more RAM
- Upgrading the CPU
- Increasing storage capacity
- Replacing the server with a more powerful machine

### Advantages

- Simple to implement
- Does not require a load balancer
- Fast communication between processes on the same machine
- Easier data consistency because there is only one server instance

### Limitations

- Hardware upgrades have a maximum limit
- The server remains a single point of failure
- High-end hardware can become expensive

## Horizontal Scaling

Horizontal scaling, also called **scaling out**, adds more servers to the system. A **load balancer** distributes incoming requests among the available servers.

### Advantages

- Capacity can grow by adding more servers
- More resilient to failures
- If one server fails, other servers can continue handling traffic
- Avoids depending entirely on a single machine

### Limitations

- Requires a load balancer and more complex infrastructure
- Maintaining consistent data across servers is harder
- Communication between servers uses network calls, which are slower than communication within one machine

## Vertical vs. Horizontal Scaling

| Feature | Vertical Scaling | Horizontal Scaling |
| --- | --- | --- |
| Approach | Upgrade one server | Add more servers |
| Load balancer | Usually not required | Required to distribute traffic |
| Failure resilience | Lower | Higher |
| Data consistency | Easier | More challenging |
| Communication | Fast internal communication | Slower network communication |
| Capacity limit | Restricted by hardware limits | Can expand by adding servers |
| Complexity | Lower | Higher |

## Hybrid Scaling

Real-world systems often combine both approaches:

- **Vertical scaling** provides simplicity, speed, and easier consistency.
- **Horizontal scaling** provides resilience and the ability to handle continued growth.

For example, a system may use several reasonably powerful servers behind a load balancer instead of relying on either one extremely powerful server or many underpowered servers.

## Key Takeaway

There is no universally better scaling method. The correct choice depends on traffic, reliability requirements, budget, system architecture, and data-consistency needs. Most large systems use a hybrid strategy to balance these trade-offs.
