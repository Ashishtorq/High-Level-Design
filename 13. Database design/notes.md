# Database Selection in System Design

Choosing a database means matching the storage system to your **data, queries, and non-functional requirements** such as latency, availability, scalability, and cost.

## 1. Key Factors in Database Choice

### Data Structure

- **Structured data:** Predictable fields and relationships, such as customers, orders, and payments. Relational databases are often a good fit.
- **Semi-structured data:** Documents with nested or varying attributes, such as product catalogs. Document databases may fit naturally.
- **Unstructured content:** Images, videos, and documents. Object storage usually holds the files, while a database stores their metadata and object keys.

Data format alone does not decide the database. Relational databases can also store JSON, and NoSQL databases still require thoughtful data modeling.

### Query Patterns

Start with the operations the application must support:

- Do queries need joins across related entities?
- Must several updates succeed or fail together in a transaction?
- Are most requests simple lookups by a key?
- Do you need full-text search, filtering, or autocomplete?
- Are you serving individual records or analyzing millions of records?

Design the schema and indexes around these access patterns.

### Scale and Reliability

Estimate data size, growth, read/write traffic, peak load, and acceptable response time. Also decide how much stale data or downtime the application can tolerate.

**SQL is not limited to small systems, and NoSQL is not automatically better at large scale.** Relational systems can scale through techniques such as indexing, replication, partitioning, and sharding. The right choice depends on the workload and operational trade-offs.

## 2. SQL vs. NoSQL

| Database type | Common fit | Examples | Main consideration |
| --- | --- | --- | --- |
| Relational | Related entities, joins, constraints, transactional workflows | PostgreSQL, MySQL | Schema, indexes, and scaling strategy |
| Document | Nested records and varied attributes | MongoDB | Embedding versus references and update patterns |
| Key-value | Fast lookup by a known key, caching, session data | Redis | Limited query flexibility compared with richer models |
| Wide-column | Large distributed datasets with predictable access patterns | Cassandra | Partition-key design and query-driven modeling |

### Relational Databases and ACID

Relational databases are a common choice for orders, payments, and inventory because they support transactions and data constraints.

**ACID** means:

- **Atomicity:** All operations in a transaction succeed, or none take effect.
- **Consistency:** A transaction preserves the database's defined rules and constraints.
- **Isolation:** Concurrent transactions interact according to the chosen isolation level.
- **Durability:** Committed changes survive failures within the database's configured guarantees.

ACID consistency concerns valid database state; it is different from consistency between distributed replicas.

### NoSQL Clarifications

NoSQL covers several database models with different guarantees. It does **not** mean transactions are unavailable: MongoDB, for example, supports multi-document transactions. Their cost and limitations still matter. [1]

Cassandra is a **wide-column database**, not the same category as an analytical column-oriented database. Its partitioned model is designed for distributed workloads, with data organized around expected queries. [2]

## 3. Specialized Storage Solutions

### Caching: Redis

Cache frequently accessed data or reusable results from expensive database queries and remote service calls.

A common pattern is **cache-aside**:

1. Check the cache.
2. On a cache miss, read from the original source.
3. Cache the result with an appropriate expiration time.
4. Return the result.

Plan for stale data, invalidation, eviction, and cache outages. If many requests miss at once, they can overload the original source.

### Object Storage and CDN

Use object storage, such as **Amazon S3**, for images, videos, PDFs, and other large files. Store file metadata and object keys in the database.

A **Content Delivery Network (CDN)** caches eligible content closer to users, reducing latency and traffic to the origin. Object storage holds the content; the CDN improves delivery.

### Search Engines: Elasticsearch

Use a search engine for features such as full-text search, relevance ranking, fuzzy matching, and autocomplete.

In a typical application, the primary database remains the source of truth and a search index contains a searchable copy. Plan how changes reach the index and how much indexing delay is acceptable.

### Data Warehouses

Use a warehouse for large analytical queries, historical reporting, and aggregations across business data.

- **OLTP:** Short operational transactions, such as creating an order.
- **OLAP:** Analytical queries, such as calculating annual sales by region.

Separating these workloads can prevent heavy reports from slowing customer-facing operations.

## 4. E-commerce Example from the Video

The supplied summary describes an Amazon-style example that combines storage technologies. Treat this as an illustrative design, not a verified description of Amazon's actual architecture.

| Data or workload | Possible storage choice |
| --- | --- |
| Active orders and payment records | Relational database for transactions and constraints |
| Historical orders queried by customer or order key | Cassandra, if the access patterns justify it |
| Frequently requested data | Redis cache |
| Product images and videos | Object storage with a CDN |
| Product search | Search engine |
| Business reports | Data warehouse |

The proposed approach moves older orders from the operational database to a historical store using scheduled jobs.

**Practical additions:**

- Historical data does not automatically require Cassandra. Relational partitions, object storage, or a warehouse may be a better fit.
- Copy and verify records before deleting them from the source.
- Make transfer jobs safe to retry without creating duplicate records.
- Account for late updates, refunds, and queries spanning current and historical orders.
- Scheduled batches suit periodic transfers; change data capture may suit ongoing synchronization.

## 5. A Simple Selection Checklist

1. Define the main entities and relationships.
2. List the important queries and transaction boundaries.
3. Estimate data volume, growth, and peak read/write load.
4. Set latency, availability, and consistency requirements.
5. Evaluate cost, backups, recovery, and team expertise.
6. Start with the simplest suitable design and add specialized stores when a clear need appears.

## Key Takeaway

**Choose databases based on access patterns and required guarantees.** A system can combine several storage technologies, but each extra store adds synchronization and operational work.

## References for Technical Clarifications

1. [MongoDB: Transactions and Operations](https://www.mongodb.com/docs/v8.0/core/transactions-operations/)
2. [Apache Cassandra: Architecture Overview](https://cassandra.apache.org/doc/stable/cassandra/architecture/overview.html)
