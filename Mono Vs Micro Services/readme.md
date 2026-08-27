# Monolith vs Microservices

## How to Convert a Monolithic Application to a Microservices Architecture

Based on the video **"Monolith vs Microservices: How to convert a monolith application to Microservice application"** by **designKarle - System Design By Shivam Tiwari**.

---

# 1. Why This Topic Matters

A lot of developers learn microservices as a list of benefits:

- independent deployment
- independent scaling
- fault isolation
- separate databases
- different technology stacks

That makes microservices sound like an obvious upgrade from a monolith.

They are not.

Microservices solve certain organizational and scaling problems, but they introduce a different class of complexity:

- network failures
- distributed transactions
- eventual consistency
- service discovery
- observability
- deployment coordination
- schema evolution
- retries
- idempotency
- duplicate messages
- debugging across services
- infrastructure overhead

The important system design question is therefore not:

> Are microservices better than monoliths?

The useful question is:

> At what point does the complexity of the monolith become more expensive than the complexity of distributed services?

This chapter explains both architectures and, more importantly, how to migrate from one to the other safely.

---

# 2. Monolithic Architecture

A monolith is an application in which multiple business capabilities are packaged and deployed together.

For example, imagine an online learning platform with the following modules:

```text
Application
├── Authentication
├── User Management
├── Course Management
├── Cart
├── Payment
├── Orders
└── Notifications
```

All of these modules may live inside one codebase.

A simplified Node.js application could look like this:

```text
src/
├── auth/
├── users/
├── courses/
├── cart/
├── payments/
├── orders/
├── notifications/
└── index.js
```

The entire application starts from one entry point.

```text
Client
   |
   v
Load Balancer
   |
   v
Monolithic Application
   |
   v
Shared Database
```

Every business module accesses the same deployment unit and often the same database.

---

# 3. Characteristics of a Monolith

## 3.1 Single Deployment Unit

The entire application is built and deployed together.

Suppose only the payment module changes.

Even then, the deployment artifact may contain:

```text
Auth
Course
Cart
Payment
Order
Notification
```

All of these are redeployed together.

---

## 3.2 Shared Runtime

Modules usually execute inside the same process.

For example:

```text
Node.js Process
|
├── Auth APIs
├── Course APIs
├── Cart APIs
├── Payment APIs
└── Order APIs
```

If the process crashes, every module becomes unavailable.

---

## 3.3 Shared Database

A common monolithic database may look like:

```text
Database
├── users
├── courses
├── carts
├── payments
├── orders
└── notifications
```

Different modules can freely join tables.

For example:

```sql
SELECT *
FROM orders
JOIN users ON orders.user_id = users.id
JOIN payments ON payments.order_id = orders.id;
```

This makes development convenient.

But it also increases coupling.

---

# 4. Advantages of a Monolith

A monolith is not inherently bad.

For many systems, especially early-stage products, a monolith is the correct architecture.

## Simple Development

Everything is in one repository and one runtime.

Developers can easily:

- navigate the codebase
- call functions directly
- reuse libraries
- debug locally

---

## Simple Deployment

Deployment may be:

```text
git push
   |
   v
CI/CD
   |
   v
Docker Image
   |
   v
Production Server
```

Only one application must be deployed.

---

## Easy Transactions

Since modules share one database, ACID transactions are straightforward.

Example:

```text
Create Order
     |
     v
Deduct Inventory
     |
     v
Create Payment Record
```

All operations can happen inside one database transaction.

```sql
BEGIN;

INSERT INTO orders ...;

UPDATE inventory ...;

INSERT INTO payments ...;

COMMIT;
```

If something fails:

```sql
ROLLBACK;
```

Distributed systems make this much harder.

---

## Easy Debugging

A request usually flows through one application.

```text
HTTP Request
    |
    v
Controller
    |
    v
Service
    |
    v
Database
```

Logs are often available in one place.

---

# 5. Problems That Appear as the Monolith Grows

The problem is usually not the monolithic architecture itself.

The problem appears when:

- traffic grows
- the engineering organization grows
- modules have very different scaling requirements
- deployment frequency increases
- the codebase becomes tightly coupled

---

# 6. Problem 1: Single Point of Failure

Suppose the application contains:

```text
Authentication
Course
Cart
Payment
Order
```

All modules run in one application process.

If a severe memory leak happens in Payment:

```text
Payment memory leak
        |
        v
Application process crashes
        |
        v
Everything becomes unavailable
```

Users can no longer:

- log in
- browse courses
- view carts
- make payments
- view orders

A failure in one business capability affects the whole system.

---

# 7. Problem 2: Inefficient Scaling

Imagine traffic distribution:

```text
Authentication: 10%
Courses:        25%
Cart:           15%
Payment:        40%
Orders:         10%
```

Payment is receiving much more load.

With a monolith, you cannot easily scale only payment.

You have to scale the entire application.

```text
            Load Balancer
            /     |     \
           /      |      \
          v       v       v
      Monolith Monolith Monolith
```

Every instance contains:

```text
Auth
Course
Cart
Payment
Orders
```

Even though Payment is the bottleneck.

This wastes resources.

---

# 8. Problem 3: Deployment Coupling

Suppose a developer changes only:

```text
Payment Service Logic
```

The pipeline may still execute:

```text
Build entire application
        |
        v
Run all tests
        |
        v
Deploy full application
```

A small change creates a large deployment blast radius.

---

# 9. Problem 4: Large Team Coordination

Imagine 100 developers working on one repository.

Teams might be:

```text
Team A -> Authentication
Team B -> Courses
Team C -> Cart
Team D -> Payments
Team E -> Orders
```

But all of them modify the same application.

This can create:

- merge conflicts
- deployment coordination
- shared release schedules
- accidental dependencies
- ownership confusion

Microservices try to align system boundaries with team boundaries.

---

# 10. Microservices Architecture

Microservices split the application into independently deployable services.

Instead of:

```text
Monolithic Application
├── Auth
├── Course
├── Cart
├── Payment
└── Order
```

we create:

```text
Auth Service
Course Service
Cart Service
Payment Service
Order Service
```

Each service:

- owns a specific business capability
- exposes an API
- can be deployed independently
- can scale independently

---

# 11. High-Level Microservices Architecture

```text
                    Client
                      |
                      v
                 API Gateway
                      |
       ---------------------------------
       |        |        |       |      |
       v        v        v       v      v
     Auth    Course    Cart   Payment  Order
   Service   Service  Service  Service Service
```

Each service may have its own datastore.

```text
Auth Service ------ Auth DB

Course Service ---- Course DB

Cart Service ------ Cart DB

Payment Service --- Payment DB

Order Service ----- Order DB
```

This is known as:

> Database per Service

---

# 12. Why Database per Service Matters

A service should own both:

1. business logic
2. business data

Suppose Payment Service and Order Service share a database.

Then Payment might directly query:

```sql
SELECT * FROM orders;
```

Now Payment depends on the Order schema.

If Order changes its schema, Payment may break.

That defeats service independence.

Instead:

```text
Payment Service
      |
      v
Payment Database
```

and:

```text
Order Service
      |
      v
Order Database
```

If Payment needs order information, it calls Order Service.

```text
Payment Service
      |
      | HTTP / gRPC
      v
Order Service
```

The database becomes an implementation detail of each service.

---

# 13. Independent Scaling

Microservices allow scaling specific workloads.

Suppose Payment receives heavy load.

Instead of:

```text
Monolith x 10
```

we can run:

```text
Auth Service x 2

Course Service x 3

Cart Service x 3

Payment Service x 10

Order Service x 3
```

Resources match actual demand.

---

# 14. Independent Deployment

Suppose Payment Service changes.

Only Payment needs deployment.

```text
Payment Code Change
       |
       v
Payment CI/CD
       |
       v
Payment Service Deployment
```

Auth, Course, Cart, and Order remain untouched.

This significantly reduces the deployment blast radius.

---

# 15. Fault Isolation

Suppose Notification Service fails.

Ideally:

```text
Notification Service DOWN
```

but:

```text
Auth    -> Working
Course  -> Working
Cart    -> Working
Payment -> Working
Order   -> Working
```

Users may still complete purchases.

Notifications can be retried later.

This is called:

> Fault isolation

Note that microservices do not automatically provide fault isolation.

Poor service design can still create cascading failures.

---

# 16. Microservices Introduce Network Calls

Inside a monolith:

```text
OrderService -> PaymentService
```

may simply be:

```javascript
paymentService.processPayment(order);
```

This is an in-memory function call.

In microservices:

```text
Order Service
      |
      v
HTTP / gRPC
      |
      v
Payment Service
```

Network communication can fail.

Possible failures include:

- timeout
- DNS failure
- connection reset
- service unavailable
- partial response
- duplicate request
- slow response

Distributed systems must assume the network is unreliable.

---

# 17. Synchronous Communication

One service can directly call another.

Example:

```text
Order Service
      |
      | HTTP
      v
Payment Service
```

Example endpoint:

```http
POST /payments
```

Request:

```json
{
  "orderId": "123",
  "amount": 2500
}
```

Response:

```json
{
  "paymentId": "pay_456",
  "status": "SUCCESS"
}
```

Common technologies:

- REST
- gRPC
- GraphQL

---

# 18. Problems With Synchronous Communication

Imagine:

```text
Order
  |
  v
Payment
  |
  v
Inventory
  |
  v
Notification
```

If every service waits for another service, latency accumulates.

Example:

```text
Order       50 ms
Payment    200 ms
Inventory  100 ms
Notification 150 ms
```

Total latency can become roughly:

```text
500+ ms
```

More importantly, if Notification fails, should the whole order fail?

Usually not.

This is where asynchronous communication becomes valuable.

---

# 19. Asynchronous Communication

Services can communicate through a message broker.

Examples:

- Kafka
- RabbitMQ
- AWS SQS
- Google Pub/Sub
- Apache Pulsar

Architecture:

```text
Order Service
      |
      | OrderCreated
      v
Message Broker
    /      \
   v        v
Payment  Notification
Service     Service
```

The Order Service publishes an event:

```json
{
  "event": "OrderCreated",
  "orderId": "123",
  "userId": "456",
  "amount": 2500
}
```

Other services react independently.

---

# 20. Event-Driven Architecture

Instead of asking another service to perform work directly, services publish facts.

Example event:

```text
OrderCreated
```

Payment Service receives it:

```text
OrderCreated
     |
     v
Payment Service
```

Inventory Service receives it:

```text
OrderCreated
     |
     v
Inventory Service
```

Notification Service receives it:

```text
OrderCreated
     |
     v
Notification Service
```

The services become less tightly coupled.

---

# 21. Microservices Are Not Automatically Better

A small startup with:

- 5 developers
- 10,000 users
- one product
- moderate traffic

probably does not need 20 microservices.

A well-structured modular monolith is often better.

Example:

```text
Application
├── Auth Module
├── Course Module
├── Cart Module
├── Payment Module
└── Order Module
```

Modules should still maintain clean boundaries even if deployed together.

This architecture is often called:

> Modular Monolith

A modular monolith can later be decomposed much more easily.

---

# 22. When Should You Consider Microservices?

Common signals include:

## Different Scaling Requirements

Example:

```text
Search traffic >> Billing traffic
```

Scaling the whole application is wasteful.

---

## Independent Teams

Different teams need independent ownership.

```text
Team Payments -> Payment Service

Team Identity -> Auth Service

Team Orders -> Order Service
```

---

## Deployment Bottlenecks

A small code change requires:

- huge test suite
- long deployment cycle
- coordination across teams

---

## Reliability Isolation

Certain workloads must not affect others.

Example:

```text
Recommendation engine failure
```

should not break:

```text
checkout
```

---

# 23. The Wrong Way: Big Bang Migration

One risky strategy is:

```text
Monolith
   |
   v
Rewrite Everything
   |
   v
Microservices
```

This creates enormous risk.

Problems include:

- long migration time
- feature freeze
- duplicated functionality
- hidden business logic
- difficult rollback
- uncertain performance characteristics

Instead, migrate gradually.

---

# 24. Strangler Fig Pattern

The recommended strategy is the:

> Strangler Fig Pattern

The idea comes from strangler fig trees that gradually grow around an existing tree and eventually replace it.

In software:

```text
Old Monolith
     |
     | gradually replaced
     v
Microservices
```

The migration happens one domain at a time.

---

# 25. Initial Architecture

Suppose we start with:

```text
                Client
                  |
                  v
              Monolith
                  |
                  v
               Database
```

Inside the monolith:

```text
Auth
Course
Cart
Payment
Order
```

---

# 26. Step 1: Identify Service Boundaries

Do not randomly split code.

Identify business capabilities.

Possible domains:

```text
Identity
Catalog
Cart
Checkout
Payments
Orders
Notifications
```

These boundaries are often called:

> Bounded Contexts

The concept comes from Domain-Driven Design.

---

# 27. What Is a Bounded Context?

A bounded context represents a clear business responsibility.

For example:

```text
Payment Context
```

might contain:

```text
Payment
Refund
Transaction
Payment Method
Gateway Integration
```

While:

```text
Order Context
```

contains:

```text
Order
Order Item
Order Status
Order History
```

Even though these domains interact, their business rules are different.

---

# 28. Choosing the First Service to Extract

Good first candidates are usually modules that:

- have clear boundaries
- have limited dependencies
- have independent scaling needs
- create operational pain

Examples:

```text
Notifications
Search
Image Processing
Payments
```

Notifications are often easier than extracting a deeply connected domain like Orders.

---

# 29. Step 2: Introduce a Routing Layer

Place an API Gateway or reverse proxy in front of the application.

Before:

```text
Client
   |
   v
Monolith
```

After:

```text
Client
   |
   v
API Gateway
   |
   v
Monolith
```

Initially, every request still goes to the monolith.

The gateway gives us a place to gradually redirect routes.

---

# 30. Step 3: Extract One Service

Suppose we choose Payment.

Before:

```text
Monolith
├── Auth
├── Course
├── Cart
├── Payment
└── Order
```

Create:

```text
Payment Service
```

Now temporarily both exist:

```text
Monolith Payment Module

and

Payment Microservice
```

This duplication is part of migration.

---

# 31. Step 4: Route Payment Traffic

The gateway can route requests based on path.

```text
/api/auth/*      -> Monolith
/api/courses/*   -> Monolith
/api/cart/*      -> Monolith
/api/orders/*    -> Monolith
/api/payments/*  -> Payment Service
```

Architecture:

```text
                 Client
                   |
                   v
              API Gateway
               /        \
              /          \
             v            v
        Monolith     Payment Service
```

---

# 32. Gradual Traffic Migration

Do not necessarily send 100% traffic immediately.

You can begin with:

```text
1%
```

then:

```text
5%
```

then:

```text
25%
```

then:

```text
50%
```

then:

```text
100%
```

This is similar to a canary deployment.

---

# 33. Canary Release

Suppose:

```text
Payment traffic = 10,000 requests/minute
```

Initially:

```text
99% -> old payment module
1%  -> new Payment Service
```

Monitor:

- error rate
- latency
- payment success rate
- database errors
- CPU
- memory

If healthy:

```text
90% -> old
10% -> new
```

Eventually:

```text
0%   -> old
100% -> new
```

---

# 34. Database Migration

This is often the hardest part.

Initially:

```text
Monolith
   |
   v
Shared Database
```

The Payment module may use tables such as:

```text
payments
transactions
refunds
```

The new service ideally owns a separate Payment DB.

```text
Payment Service
      |
      v
Payment Database
```

But migration creates a transition problem.

---

# 35. Transitional Database Architecture

During migration:

```text
Monolith -------- Shared DB

Payment Service - Payment DB
```

Now two systems may need payment-related information.

This introduces:

> Data synchronization

---

# 36. Dual Write Strategy

One strategy is writing to both databases.

```text
Payment Request
      |
      v
Payment Logic
    /     \
   v       v
Old DB   New DB
```

Problem:

What happens if:

```text
Old DB write succeeds
```

but:

```text
New DB write fails?
```

Now the databases disagree.

Dual writes are deceptively difficult.

---

# 37. Better Pattern: Transactional Outbox

A safer pattern is:

> Transactional Outbox

Suppose the monolith updates the payment table.

Inside the same database transaction:

```text
payments table
+
outbox table
```

Example:

```sql
BEGIN;

UPDATE payments
SET status = 'SUCCESS'
WHERE id = 100;

INSERT INTO outbox_events (
  type,
  payload
)
VALUES (
  'PaymentUpdated',
  '{...}'
);

COMMIT;
```

A background worker publishes the outbox event.

```text
Outbox Table
     |
     v
Publisher
     |
     v
Kafka
     |
     v
Payment Service
```

This avoids inconsistent dual writes.

---

# 38. Change Data Capture

Another technique is:

> Change Data Capture (CDC)

Tools like Debezium can observe database changes.

Architecture:

```text
Monolith DB
    |
    | transaction log
    v
Debezium
    |
    v
Kafka
    |
    v
Payment Service
```

The new service receives changes and builds its own database.

---

# 39. Backfilling Historical Data

A new service may need existing records.

Suppose the old database contains:

```text
10 million payments
```

You need to copy them.

Typical strategy:

```text
Historical Data
      |
      v
Batch Migration
      |
      v
New Payment DB
```

Meanwhile, new changes are streamed through CDC or events.

Eventually:

```text
Old data + new updates
```

converge in the new database.

---

# 40. Data Validation

Before switching traffic, compare both systems.

Example metrics:

```text
Old payment count = 10,342,881

New payment count = 10,342,881
```

Compare:

- row counts
- payment totals
- statuses
- timestamps
- reconciliation reports

Data migration should be verified, not assumed.

---

# 41. Read Migration

Sometimes migration occurs in phases.

Phase 1:

```text
Writes -> Old DB
Reads  -> Old DB
```

Phase 2:

```text
Writes -> Old DB + sync
Reads  -> New DB for shadow testing
```

Phase 3:

```text
Reads -> New DB
Writes -> transition mechanism
```

Phase 4:

```text
Reads  -> New DB
Writes -> New DB
```

---

# 42. Shadow Traffic

The new service can process production requests without affecting users.

Example:

```text
User Request
     |
     v
Old Payment System ----> actual response
     |
     +---- copy ----> New Payment Service
```

The new service response is ignored.

Compare:

```text
Old result
vs
New result
```

This helps detect behavioral differences.

---

# 43. Service-to-Service Communication

After extracting Payment Service, the monolith may still need payment operations.

Old:

```text
Order Module
    |
    v
Payment Module Function
```

New:

```text
Order Module
    |
    v
HTTP/gRPC
    |
    v
Payment Service
```

The migration has converted a local function call into a network dependency.

This means new failure cases must be handled.

---

# 44. Timeouts

Never wait indefinitely for another service.

Bad:

```javascript
await callPaymentService();
```

without a timeout.

Better:

```text
Payment Service timeout = 2 seconds
```

If no response arrives, fail predictably.

---

# 45. Retries

Some failures are temporary.

Example:

```text
503 Service Unavailable
```

The caller may retry.

But retries should use:

> Exponential Backoff

Example:

```text
1st retry -> 100ms
2nd retry -> 200ms
3rd retry -> 400ms
4th retry -> 800ms
```

Often with random jitter.

---

# 46. Retry Storms

Imagine 10,000 clients retry immediately.

```text
Service fails
     |
     v
10,000 retries
     |
     v
Service receives even more traffic
```

This can make the outage worse.

Use:

- exponential backoff
- jitter
- retry limits

---

# 47. Idempotency

Payment operations must not accidentally execute twice.

Suppose:

```text
Client -> Payment Service
```

Payment succeeds.

But the response is lost.

The client retries.

Without protection:

```text
Customer charged twice
```

Use an idempotency key.

```http
Idempotency-Key: order_123_payment
```

The service stores the result.

If the same request arrives again:

```text
Return original result
```

instead of executing another payment.

---

# 48. Circuit Breaker

Suppose Payment Service is failing.

Without protection:

```text
Order Service
   |
   | repeated requests
   v
Payment Service
```

The failure can consume:

- threads
- connections
- CPU
- memory

A circuit breaker temporarily stops calls.

States:

```text
CLOSED
  |
  | failures exceed threshold
  v
OPEN
  |
  | wait
  v
HALF-OPEN
```

If recovery succeeds:

```text
CLOSED
```

---

# 49. Distributed Transactions

In a monolith:

```text
BEGIN TRANSACTION
```

can coordinate multiple tables.

In microservices:

```text
Order DB
Payment DB
Inventory DB
```

No single database transaction spans everything safely.

Example workflow:

```text
Create Order
     |
     v
Reserve Inventory
     |
     v
Charge Payment
```

What if:

```text
Inventory succeeds
Payment fails
```

The system must compensate.

---

# 50. Saga Pattern

A Saga represents a distributed business transaction as multiple local transactions.

Example:

```text
Create Order
     |
     v
Reserve Inventory
     |
     v
Charge Payment
```

If payment fails:

```text
Release Inventory
     |
     v
Cancel Order
```

These reverse actions are called:

> Compensating Transactions

---

# 51. Choreography-Based Saga

Services communicate using events.

```text
Order Service
    |
    | OrderCreated
    v
Kafka
    |
    v
Inventory Service
    |
    | InventoryReserved
    v
Kafka
    |
    v
Payment Service
```

Failure:

```text
PaymentFailed
      |
      v
Kafka
      |
      v
Inventory releases stock
```

No central controller exists.

---

# 52. Orchestration-Based Saga

A central orchestrator controls the workflow.

```text
Saga Orchestrator
      |
      +--> Order Service
      |
      +--> Inventory Service
      |
      +--> Payment Service
```

The orchestrator decides the next step.

Example:

```text
createOrder()
reserveInventory()
chargePayment()
```

Failure:

```text
refundPayment()
releaseInventory()
cancelOrder()
```

---

# 53. Eventual Consistency

In distributed systems, different services may temporarily disagree.

Example:

Immediately after checkout:

```text
Order Service: PAID

Analytics Service: PAYMENT_PENDING
```

After the event is processed:

```text
Analytics Service: PAID
```

The system eventually converges.

This is:

> Eventual Consistency

---

# 54. API Gateway

An API Gateway is commonly used as the public entry point.

```text
                    Client
                      |
                      v
                 API Gateway
                      |
        ------------------------------
        |       |       |      |     |
        v       v       v      v     v
       Auth   Course   Cart Payment Order
```

Responsibilities may include:

- routing
- authentication
- rate limiting
- request aggregation
- TLS termination
- logging
- versioning

---

# 55. API Gateway vs Load Balancer

These are related but not identical.

A load balancer distributes traffic across replicas.

```text
             Load Balancer
           /      |       \
          v       v        v
       Payment  Payment  Payment
       Instance Instance Instance
```

An API Gateway decides which service should receive a request.

```text
/api/orders   -> Order Service
/api/payment  -> Payment Service
/api/users    -> User Service
```

In many architectures:

```text
Client
  |
  v
API Gateway
  |
  v
Load Balancer
  |
  v
Service Instances
```

Cloud platforms may combine some of these responsibilities.

---

# 56. Service Discovery

Suppose Payment Service has multiple instances:

```text
payment-1
payment-2
payment-3
```

Other services need a way to locate them.

Service discovery systems include:

- Kubernetes Services
- Consul
- AWS Cloud Map
- Eureka

Example:

```text
payment-service.internal
```

resolves to healthy instances.

---

# 57. Containerization

Microservices are commonly containerized.

Example:

```text
Auth Service     -> Docker Image
Payment Service  -> Docker Image
Order Service    -> Docker Image
```

Each image contains:

```text
application
runtime
dependencies
configuration
```

Containers improve deployment consistency.

---

# 58. Kubernetes

At larger scale, Kubernetes can orchestrate services.

Example:

```text
Kubernetes Cluster
|
├── auth-deployment
├── course-deployment
├── cart-deployment
├── payment-deployment
└── order-deployment
```

Each deployment may contain multiple pods.

```text
Payment Deployment
|
├── payment-pod-1
├── payment-pod-2
├── payment-pod-3
└── payment-pod-4
```

Kubernetes provides:

- scheduling
- service discovery
- health checks
- auto-restarts
- rolling updates
- horizontal scaling

---

# 59. Observability Becomes Critical

In a monolith, one request might produce logs in one process.

In microservices:

```text
Client
  |
  v
Gateway
  |
  v
Order
  |
  v
Payment
  |
  v
Fraud
  |
  v
Bank
```

A failure can occur anywhere.

You need:

- centralized logs
- metrics
- distributed tracing

---

# 60. Centralized Logging

Services should ship logs to a centralized platform.

Examples:

- ELK Stack
- OpenSearch
- Splunk
- Datadog
- CloudWatch

Instead of checking:

```text
server-1 logs
server-2 logs
server-3 logs
```

engineers query one system.

---

# 61. Distributed Tracing

A request receives a trace ID.

Example:

```text
Trace ID: abc123
```

The same ID propagates through:

```text
Gateway
Order Service
Payment Service
Inventory Service
Notification Service
```

Tracing tools include:

- OpenTelemetry
- Jaeger
- Zipkin
- AWS X-Ray

---

# 62. Metrics

Important service metrics include:

```text
request rate
error rate
latency
CPU
memory
queue lag
database connections
```

A useful framework is RED:

```text
R = Rate
E = Errors
D = Duration
```

---

# 63. Health Checks

Services should expose health endpoints.

Example:

```http
GET /health
```

Response:

```json
{
  "status": "UP"
}
```

Or:

```text
/health/live
/health/ready
```

Liveness asks:

> Is the process alive?

Readiness asks:

> Is it ready to receive traffic?

---

# 64. Rollback Strategy

Every migration step should support rollback.

Suppose 25% of traffic is going to the new Payment Service.

If failure rate increases:

```text
Gateway
  |
  +--> 0% New Payment Service
  |
  +--> 100% Old Monolith
```

This is one major advantage of gradual migration.

---

# 65. Feature Flags

Traffic migration can also be controlled using feature flags.

Example:

```javascript
if (featureFlags.useNewPaymentService) {
    return paymentClient.pay();
}

return legacyPayment.pay();
```

Flags allow fast rollback without a full deployment.

---

# 66. Decommissioning the Old Module

Once:

```text
100% traffic -> Payment Service
```

and the new system is stable:

1. stop writes to old payment tables
2. verify data consistency
3. remove old routing
4. remove old payment code
5. archive unused database tables
6. remove legacy monitoring

The monolith becomes smaller.

---

# 67. Repeat the Migration

After Payment:

```text
Payment -> Microservice
```

next extract another domain.

Example:

```text
Notification
```

Then:

```text
Cart
```

Then:

```text
Orders
```

Eventually:

```text
Monolith shrinks
```

while services grow around it.

---

# 68. Migration Evolution

Stage 1:

```text
[ MONOLITH ]
```

Stage 2:

```text
[ MONOLITH ] + Payment Service
```

Stage 3:

```text
[ MONOLITH ] + Payment + Notification
```

Stage 4:

```text
[ MONOLITH ] + Payment + Notification + Cart
```

Stage 5:

```text
Auth + Course + Cart + Payment + Order + Notification
```

The original monolith may eventually disappear.

But it does not have to.

Some domains may remain in a modular monolith indefinitely.

---

# 69. Common Migration Mistakes

## Splitting Services Too Small

Bad:

```text
UserNameService
UserEmailService
UserAddressService
```

This produces excessive network communication.

Service boundaries should represent meaningful business capabilities.

---

## Sharing the Same Database Forever

Architecture:

```text
Service A
Service B
Service C
    |
    v
Shared Database
```

This creates a distributed monolith.

Services are independently deployed but still tightly coupled through data.

---

## Too Many Synchronous Calls

Example:

```text
A -> B -> C -> D -> E
```

A failure in E can break the whole request.

Consider asynchronous workflows where appropriate.

---

## Ignoring Observability

Without tracing:

```text
Request failed
```

but nobody knows where.

Distributed systems without observability are extremely difficult to operate.

---

## No Idempotency

Retries are normal.

Therefore operations must tolerate duplicate requests.

This is especially important for:

- payments
- order creation
- inventory updates
- message consumers

---

# 70. Modular Monolith as a Better Starting Point

A strong architecture journey often looks like:

```text
Simple Monolith
      |
      v
Modular Monolith
      |
      v
Selective Microservices
```

rather than:

```text
New Startup
   |
   v
40 Microservices
```

The modular monolith provides clean domain boundaries without distributed-system complexity.

---

# 71. Example: E-Commerce Migration

Initial system:

```text
                Client
                  |
                  v
              Monolith
                  |
                  v
            Shared Database
```

Modules:

```text
Users
Products
Cart
Orders
Payments
Inventory
Notifications
```

---

## Stage 1: Extract Notification Service

```text
             Monolith
                 |
                 | OrderCreated
                 v
               Kafka
                 |
                 v
       Notification Service
```

This is relatively low risk.

---

## Stage 2: Extract Payment Service

```text
Client
  |
  v
Gateway
  |
  +--> Monolith
  |
  +--> Payment Service
```

Payment receives:

```text
/payments/*
```

---

## Stage 3: Extract Inventory

```text
OrderCreated
     |
     v
Inventory Service
```

Inventory owns:

```text
stock
reservations
warehouses
```

---

## Stage 4: Extract Order Service

Order now coordinates:

```text
Inventory
Payment
Notifications
```

possibly using a Saga.

---

# 72. Final Architecture Example

```text
                         Client
                           |
                           v
                      API Gateway
                           |
          ---------------------------------------
          |        |        |        |          |
          v        v        v        v          v
        Auth    Product    Cart     Order     Payment
       Service   Service  Service   Service    Service
                                      |
                                      v
                                   Kafka
                                  /     \
                                 v       v
                          Inventory   Notification
                           Service       Service
```

Databases:

```text
Auth Service         -> Auth DB

Product Service      -> Product DB

Cart Service         -> Cart DB / Redis

Order Service        -> Order DB

Payment Service      -> Payment DB

Inventory Service    -> Inventory DB

Notification Service -> Notification DB
```

---

# 73. Interview-Level Comparison

| Area | Monolith | Microservices |
|---|---|---|
| Deployment | Entire app | Individual service |
| Scaling | Whole application | Per service |
| Database | Often shared | Usually database per service |
| Transactions | Simple ACID | Distributed / Saga |
| Communication | Function calls | Network calls / events |
| Debugging | Easier | Harder |
| Infrastructure | Simple | Complex |
| Fault Isolation | Lower | Better when designed correctly |
| Team Independence | Lower | Higher |
| Operational Cost | Lower | Higher |
| Best For | Small-medium systems | Large systems / organizations |

---

# 74. Important System Design Interview Insight

Do not say:

> Microservices are more scalable, so we should use microservices.

A stronger answer is:

> I would start with a modular monolith unless we have clear requirements for independent scaling, deployment, fault isolation, or team ownership. If those pressures appear, I would extract bounded contexts incrementally using the Strangler Fig Pattern.

This demonstrates architectural maturity.

---

# 75. Migration Checklist

## Before Migration

- [ ] Identify the real bottleneck.
- [ ] Confirm microservices solve that bottleneck.
- [ ] Define bounded contexts.
- [ ] Improve observability.
- [ ] Introduce API Gateway or routing layer.
- [ ] Choose a low-risk service to extract.

## During Migration

- [ ] Create the new service.
- [ ] Define API contracts.
- [ ] Create service-owned database.
- [ ] Backfill historical data.
- [ ] Synchronize live changes.
- [ ] Add timeouts.
- [ ] Add retries with backoff.
- [ ] Add idempotency.
- [ ] Add tracing.
- [ ] Add health checks.
- [ ] Shadow production traffic.
- [ ] Canary traffic gradually.
- [ ] Monitor business metrics.
- [ ] Maintain rollback capability.

## After Migration

- [ ] Route 100% traffic.
- [ ] Validate data consistency.
- [ ] Stop legacy writes.
- [ ] Remove old module.
- [ ] Remove unused tables.
- [ ] Remove migration code.
- [ ] Document ownership.
- [ ] Repeat for another domain only when justified.

---

# 76. Key Takeaways

## 1. Microservices are not automatically better.

They trade codebase complexity for distributed-system complexity.

## 2. Start with clear domain boundaries.

A modular monolith is often the best starting architecture.

## 3. Avoid Big Bang rewrites.

Use the Strangler Fig Pattern.

## 4. Extract one service at a time.

Move traffic gradually.

## 5. Database separation is the hard part.

A microservice should eventually own its data.

## 6. Distributed transactions require new patterns.

Common approaches include:

- Saga
- Outbox
- Event-driven architecture
- Eventual consistency

## 7. Reliability must be designed explicitly.

Use:

- timeouts
- retries
- exponential backoff
- circuit breakers
- idempotency

## 8. Observability is mandatory.

Use:

- centralized logging
- metrics
- distributed tracing

## 9. Keep rollback possible.

Canary releases and feature flags reduce migration risk.

## 10. Microservices are primarily an organizational and operational architecture.

They become valuable when different parts of the system genuinely need:

- separate ownership
- separate scaling
- separate release cycles
- separate failure boundaries

---

# 77. One-Screen Mental Model

```text
MONOLITH

Client
  |
  v
Application
├── Auth
├── Cart
├── Payment
├── Order
└── Notification
  |
  v
Shared DB
```

Migration:

```text
                    API Gateway
                    /         \
                   /           \
              Monolith      Payment
                              Service
                                |
                                v
                            Payment DB
```

Continue strangling the monolith:

```text
                         API Gateway
                              |
          -----------------------------------------
          |          |         |         |        |
          v          v         v         v        v
        Auth       Cart      Order    Payment  Notification
       Service    Service   Service   Service    Service
          |          |         |         |          |
          v          v         v         v          v
       Auth DB    Cart DB   Order DB Payment DB Notification DB
```

Communication:

```text
Synchronous:
Order Service ---> Payment Service

Asynchronous:
Order Service ---> Kafka ---> Notification Service
                         ---> Inventory Service
```

Distributed workflow:

```text
Create Order
    |
Reserve Inventory
    |
Charge Payment
    |
Payment fails
    |
Release Inventory
    |
Cancel Order
```

That is the core architectural journey from a monolith to microservices.