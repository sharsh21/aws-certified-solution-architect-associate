# Database Services — SAA-C03 Exam Guide

---

## RDS — Relational Database Service

### Supported Engines
MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, IBM Db2

### Multi-AZ vs Read Replicas (CRITICAL DISTINCTION)

| | Multi-AZ | Read Replica |
|-|---------|-------------|
| **Purpose** | **High Availability (HA)** | **Performance / Scale reads** |
| **Replication** | Synchronous (standby is up-to-date) | Asynchronous (slight lag) |
| **Standby** | Cannot be read (hot standby) | Can be read (offload queries) |
| **Failover** | Automatic (1-2 min), DNS changes | Manual promotion |
| **Regions** | Same region only | Same or cross-region |
| **Count** | 1 standby | Up to 5 replicas |

> **Exam Tip:** "High availability" = Multi-AZ. "Read performance/scale" = Read Replica. Multi-AZ standby CANNOT be read directly.

### RDS Key Features
- Automated backups: 1-35 days retention, point-in-time recovery
- Manual snapshots: retained until deleted
- Storage auto-scaling: increases when free space < 10%
- **RDS Proxy:** Connection pooling, reduces DB connections, failover 66% faster
  - Required for Lambda → RDS (Lambda can create many connections)
- **IAM Authentication:** MySQL, PostgreSQL support IAM DB auth (token-based)
- Encryption at rest: KMS, must enable at creation (or restore from encrypted snapshot)
- Encryption in transit: SSL/TLS

### RDS Event Notifications
- SNS notifications for DB events (failover, backup, maintenance)
- Does NOT notify on data changes

### Real-World Use Cases & When to Choose RDS

> **The problem it solves:** Fully managed relational database — AWS handles backups, patching, replication, and failover so you can focus on your application rather than DBA work.

**Choose RDS when:** Your app needs SQL, ACID transactions, and structured relational data. You need managed HA (Multi-AZ) without building it yourself.

**Real-World Scenarios:**
| Business Problem | RDS Solution |
|-----------------|-------------|
| E-commerce platform with orders, products, users (relational model) | RDS MySQL Multi-AZ — automatic failover < 2 min if primary goes down |
| 10,000 concurrent app connections overwhelming the DB | **RDS Proxy** — pools connections, reduces DB load, Lambda → RDS pattern |
| Financial app: auditors require point-in-time recovery for transactions | RDS automated backups (35-day retention) + manual snapshots before major releases |
| Global company needs EU instance with reporting replica in US | RDS Read Replica (cross-region) — offload analytics queries from production |
| Healthcare DB must be encrypted — regulatory requirement | Enable KMS encryption at RDS creation (can't add later — must restore from encrypted snapshot) |

**Multi-AZ vs Read Replica — the most tested RDS distinction:**
- Multi-AZ = HA failover (standby can't be read) → "high availability" keyword
- Read Replica = scale reads (can be promoted for DR) → "performance" or "read scaling" keyword

---

## Aurora

### Key Differentiators vs RDS
- **MySQL and PostgreSQL compatible** (drop-in replacement)
- Storage auto-grows in 10GB increments up to **128TB**
- 6 copies of data across 3 AZs (4/6 for writes, 3/6 for reads)
- **15 Read Replicas** (vs 5 for RDS)
- Failover: < 30 seconds (automatic)
- Storage is shared across all instances (not local)
- Up to 5x faster than MySQL, 3x faster than PostgreSQL

### Aurora Architecture
- **Cluster Volume:** Shared storage, 6 copies across 3 AZs
- **Writer Endpoint:** Always points to primary
- **Reader Endpoint:** Load balances across read replicas
- **Custom Endpoint:** Route to specific subset of instances

### Aurora Serverless
| Version | Use Case |
|---------|---------|
| **v1** | Infrequent, unpredictable workloads, scales to zero |
| **v2** | Fine-grained scaling, supports more features, recommended |

- **Data API:** HTTP endpoint to run SQL (serverless, no connection management)
- Use for: dev/test, variable workloads, new apps

### Aurora Global Database
- **1 primary region** (read/write) + up to **5 secondary regions** (read only)
- Replication lag < **1 second**
- Secondary can be promoted to primary in < **1 minute** (DR)
- Write forwarding: secondary can forward writes to primary
- Use for: global low-latency reads + DR

### Aurora Multi-Master
- All instances can write (active-active)
- Used for continuous write availability (no single point of failure for writes)
- **Not commonly asked** vs Global Database

### Real-World Use Cases & When to Choose Aurora

> **The problem it solves:** Enterprise-grade relational DB with 5x MySQL / 3x PostgreSQL performance, auto-scaling storage up to 128TB, and superior HA — at a lower cost than commercial databases like Oracle.

**Choose Aurora over RDS when:** You need > 5 read replicas, storage auto-grow beyond 64TB, faster failover (< 30s vs 1-2 min for RDS), or global low-latency reads.

**Real-World Scenarios:**
| Business Problem | Aurora Solution |
|-----------------|----------------|
| SaaS platform with unpredictable DB load (spiky, sometimes zero) | **Aurora Serverless v2** — scales from 0.5 to 128 ACUs, pay per second |
| Global social network — users in US and Asia need low-latency reads | **Aurora Global Database** — primary in us-east-1, replica in ap-southeast-1, < 1s replication lag |
| E-commerce site has 50,000 products and 500 read queries/sec | Aurora with 15 Read Replicas + Reader Endpoint (load-balanced reads) |
| Enterprise moving from Oracle ($500K/yr license) to cloud | Aurora PostgreSQL — Oracle-compatible extensions, fraction of the cost |
| RPO=seconds, RTO=1 minute for DR (financial app) | Aurora Global Database — promote secondary region in < 1 minute |
| Development team needs fresh DB clones daily for testing | Aurora Fast Cloning — instant clone without duplicating data (copy-on-write) |

**Aurora vs RDS for the exam:** If the question says "MySQL/PostgreSQL compatible, high performance, auto-scaling storage, 15 replicas, or global" → Aurora. Oracle/SQL Server/Db2 → RDS only.

---

## DynamoDB (HIGH EXAM WEIGHT)

### Core Concepts
- Fully managed NoSQL (key-value + document)
- Single-digit millisecond latency at any scale
- Serverless, no maintenance
- **Tables, Items (rows), Attributes (columns)**
- Max item size: **400KB**

### Primary Keys
| Type | Description |
|------|-------------|
| **Partition Key only** | Simple PK — must be unique per item |
| **Partition Key + Sort Key** | Composite PK — PK can repeat, PK+SK must be unique |

> **Exam Tip:** Choose high-cardinality partition key (user_id, order_id) to distribute data evenly. Hot partition = performance issue.

### Read Consistency
| Type | Description | Cost |
|------|-------------|------|
| **Eventually Consistent Read** | Default, may return stale data | 1 RCU per 4KB |
| **Strongly Consistent Read** | Returns latest data, higher latency | 2 RCU per 4KB |

### Capacity Modes
| Mode | Best For |
|------|---------|
| **Provisioned (+ Auto Scaling)** | Predictable traffic, lower cost |
| **On-Demand** | Unpredictable, new tables, spiky traffic |

### DynamoDB Indexes
| Index | Type | Key Facts |
|-------|------|-----------|
| **LSI (Local Secondary Index)** | Same partition key, different sort key | Created at table creation only, up to 5 |
| **GSI (Global Secondary Index)** | Different partition key + sort key | Created anytime, own capacity, up to 20 |

> **Exam Tip:** LSI = "local" to partition, must create at table creation. GSI = "global", flexible, can add later.

### DynamoDB Accelerator (DAX)
- In-memory cache for DynamoDB
- Microsecond latency (vs millisecond)
- No application code changes (API-compatible)
- **Not suitable for strongly consistent reads**
- Write-through cache
- TTL default: 5 minutes (item cache), 10 minutes (query cache)

### DynamoDB Streams
- Ordered stream of item-level changes (CRUD)
- Data retained for **24 hours**
- Trigger Lambda, process with Kinesis
- Use cases: cross-region replication, analytics, notifications

### DynamoDB Global Tables
- Multi-region, multi-active replication
- Requires DynamoDB Streams
- All regions can read AND write
- Conflict resolution: last writer wins

### DynamoDB TTL
- Auto-delete items after timestamp
- No extra cost for deletion
- Use for session data, event logs, temporary data

### DynamoDB Transactions
- `TransactWriteItems` and `TransactGetItems`
- Up to 100 items per transaction
- All-or-nothing (ACID-like)

### Real-World Use Cases & When to Choose DynamoDB

> **The problem it solves:** Infinitely scalable, serverless NoSQL database with single-digit millisecond latency at any scale — no schema design constraints, no capacity planning, no maintenance windows.

**Choose DynamoDB when:** Scale is unpredictable or massive, you need key-value or document access patterns, or you want a truly serverless architecture.

**Real-World Scenarios:**
| Business Problem | DynamoDB Solution |
|-----------------|-----------------|
| Gaming app: 1M concurrent users store session state — must scale instantly | DynamoDB On-Demand — scales automatically, no pre-provisioning, 1ms reads |
| IoT: 10,000 sensors send readings every second (10K writes/sec) | DynamoDB with Provisioned + Auto Scaling — handles massive ingest |
| Shopping cart: each user has a cart (simple key-value, not relational) | DynamoDB — `userId` as partition key, cart items as attributes, TTL for abandoned carts |
| Real-time leaderboard needs top 100 players globally | DynamoDB with GSI (score as sort key) + **DAX** for microsecond reads |
| Multi-region app needs users in both US and EU to write locally | **DynamoDB Global Tables** — active-active replication, last-writer-wins conflict resolution |
| User session data should expire after 24 hours automatically | DynamoDB **TTL** — set expiry timestamp attribute, DynamoDB auto-deletes at no charge |
| Order processing: insert order AND decrement inventory atomically | DynamoDB **Transactions** (TransactWriteItems) — all-or-nothing across multiple items |

**When NOT DynamoDB:** Complex joins and relational queries → RDS/Aurora. Strong consistency for financial transactions at scale → Aurora. Item > 400KB → S3 + DynamoDB pointer.

---

## ElastiCache

### Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| Data structures | Strings, Lists, Sets, Sorted Sets, Hashes | Simple key-value only |
| Persistence | Yes (RDB + AOF) | No |
| Replication | Yes (Multi-AZ, read replicas) | No |
| Clustering | Yes (cluster mode) | Yes (multi-node) |
| Pub/Sub | Yes | No |
| Sorted sets (leaderboards) | Yes | No |
| Multi-AZ failover | Yes | No |
| Backup/Restore | Yes | No |

> **Exam Tip:** If question mentions **leaderboards**, **pub/sub**, **persistence**, **HA/failover** → Redis. Memcached = simple caching, multi-threaded.

### Common Caching Patterns
| Pattern | Description | Use Case |
|---------|-------------|---------|
| **Lazy Loading** | Load on cache miss | General purpose |
| **Write-Through** | Update cache on write | Data freshness critical |
| **Session Store** | Store user sessions in Redis | Stateless applications |

### ElastiCache Security
- VPC deployment
- Redis AUTH (password)
- Encryption in transit (TLS) and at rest (KMS)
- IAM auth for Redis (newer)

### Real-World Use Cases & When to Choose ElastiCache

> **The problem it solves:** In-memory cache layer in front of your database — serves repeated requests from RAM instead of hitting the DB, reducing latency from milliseconds to microseconds and cutting DB costs.

**Choose Redis when:** You need persistence, pub/sub, leaderboards (sorted sets), HA/failover, or complex data structures.
**Choose Memcached when:** Simple caching only, maximum throughput with multi-threading, and you don't need persistence or HA.

**Real-World Scenarios:**
| Business Problem | ElastiCache Solution |
|-----------------|---------------------|
| Product page loads in 800ms — DB query for product details runs every time | **Redis Lazy Loading** — cache product on first fetch, serve from cache for 5 min TTL |
| 100,000 active users on an e-commerce site — sessions stored in RDS creating bottleneck | **Redis as session store** — all app servers read/write sessions to Redis, stateless EC2s |
| Game with 1M players needs real-time leaderboard (top 100 scores) | **Redis Sorted Sets** — O(log n) insert/query, perfect for ranked leaderboards |
| Pub/Sub: order service must notify 5 microservices when an order is placed | **Redis Pub/Sub** — order service publishes, subscribers receive instantly |
| Write-heavy app: every product update must reflect immediately in cache | **Redis Write-Through** — update cache on every write, never stale data |
| Simple caching for a PHP app — no HA needed, max throughput | **Memcached** — multi-threaded, horizontally scalable, zero persistence overhead |

**Exam keyword triggers:** "leaderboard" → Redis (Sorted Sets). "pub/sub" → Redis. "session store" → Redis. "simple caching, no HA" → Memcached.

---

## Redshift

- Fully managed **data warehouse** (OLAP, not OLTP)
- Columnar storage, parallel query execution
- Up to petabyte scale
- SQL-based (PostgreSQL-compatible)
- **Leader Node:** Query planning and coordination
- **Compute Nodes:** Data storage and query execution

### Redshift Key Features
- **Redshift Spectrum:** Query S3 data without loading into Redshift
- **AQUA (Advanced Query Accelerator):** Hardware-accelerated cache
- **Redshift Serverless:** Auto-scales, no provisioning
- **Snapshots:** Automated or manual, cross-region copy
- **Enhanced VPC Routing:** Data through VPC (for security)

> **Exam Tip:** Redshift = analytics/reporting. RDS = transactional. If question says "data warehouse" or "OLAP" → Redshift.

### Real-World Use Cases & When to Choose Redshift

> **The problem it solves:** Petabyte-scale data warehouse for analytics and reporting — columnar storage and MPP (Massively Parallel Processing) make it dramatically faster than running OLAP queries on a transactional RDS database.

**Choose Redshift when:** You need to run complex analytical queries across billions of rows (OLAP), build BI dashboards, or query historical data.

**Real-World Scenarios:**
| Business Problem | Redshift Solution |
|-----------------|-----------------|
| Retail company needs to analyze 3 years of sales data (10B rows) for trends | Redshift — columnar storage + MPP returns query in seconds vs hours on MySQL |
| BI team runs QuickSight dashboards — queries take 10 minutes on RDS | Migrate to Redshift — 10x faster for OLAP, connect QuickSight to Redshift |
| Data lake in S3 (100TB Parquet) — analysts want SQL access without loading into DB | **Redshift Spectrum** — query S3 directly with SQL, no data movement |
| Company needs zero-management data warehouse (no cluster sizing) | **Redshift Serverless** — auto-scales, pay per query, no cluster management |
| Analytics workload is bursty — big reports on Monday, idle rest of week | Redshift with pause/resume — stop cluster when not in use, save costs |

**The OLTP vs OLAP distinction (critical for exam):** RDS/Aurora = OLTP (many small transactions). Redshift = OLAP (few large analytical queries). Never use RDS for a data warehouse question.

---

## DMS — Database Migration Service

- Migrate databases to AWS
- Source stays operational during migration (minimal downtime)
- **SCT (Schema Conversion Tool):** Convert schema for heterogeneous migrations
- **Continuous Replication (CDC):** Keep source and target in sync
- **Homogeneous:** Same engine (Oracle → RDS Oracle) — no SCT needed
- **Heterogeneous:** Different engine (Oracle → Aurora) — SCT required

---

## Other Databases

### DocumentDB
- MongoDB-compatible managed document database
- Not a MongoDB installation — it's Aurora's MongoDB-compatible API
- Auto-scaling storage, Multi-AZ, 6 copies

### Neptune
- Fully managed **graph database**
- Property Graph (Gremlin) and RDF (SPARQL) support
- Use for: social networks, recommendation engines, fraud detection, knowledge graphs
- Multi-AZ, up to 15 read replicas

> **Real-World:** LinkedIn uses graph DBs to power "People You May Know" — each person is a node, relationships are edges. Neptune answers "find all users within 2 degrees of connection" in milliseconds. Fraud detection: "find all accounts that share the same device, phone, and IP" → graph traversal vs SQL joins.

### Keyspaces (Apache Cassandra)
- Managed Cassandra-compatible service
- CQL (Cassandra Query Language)
- Use for: existing Cassandra workloads on AWS

> **Real-World:** Netflix uses Cassandra for viewing history (write-heavy, time-series-like, massive scale). Keyspaces lets you migrate existing Cassandra apps to AWS without changing application code.

### Timestream
- Fully managed **time-series database**
- IoT applications, operational metrics, telemetry
- Fast ingestion, SQL queries

> **Real-World:** Industrial IoT — 10,000 temperature sensors send readings every second. Timestream auto-tiers recent data (fast storage) vs historical (cost-effective storage). Query: "average temperature per sensor over last 7 days" runs in seconds vs minutes on DynamoDB.

### QLDB — Quantum Ledger Database
- Immutable, cryptographically verifiable transaction log
- Use for: financial transactions, supply chain, audit history
- **Not decentralized** (unlike blockchain) — owned by central authority

> **Real-World:** Bank needs an immutable audit log of every transaction — regulators can cryptographically verify that no record was altered. QLDB provides a ledger where you can prove "this $10,000 transfer happened at exactly 14:32:05 and was never modified."

### Managed Blockchain
- Create and manage blockchain networks
- Hyperledger Fabric, Ethereum
- Decentralized — no central authority

> **Real-World:** Supply chain consortium (Walmart + suppliers) — each participant validates goods transfer. No single party controls the ledger. Unlike QLDB (central authority), Managed Blockchain is for multi-party trust with no central owner.

---

## Database Selection Guide (Exam Cheat Sheet)

| Use Case | Service |
|---------|---------|
| OLTP (MySQL, PostgreSQL) | RDS or Aurora |
| OLTP (Oracle, SQL Server) | RDS |
| Global OLTP, auto-scaling storage, 15 replicas | Aurora |
| NoSQL, any scale, serverless | DynamoDB |
| In-memory cache, leaderboards, pub/sub | ElastiCache Redis |
| In-memory cache, simple, multi-threaded | ElastiCache Memcached |
| Data warehouse, OLAP, analytics | Redshift |
| Graph relationships | Neptune |
| MongoDB-compatible | DocumentDB |
| Time series data, IoT | Timestream |
| Immutable audit ledger | QLDB |
| Cassandra workloads | Keyspaces |

---

## Key Exam Traps

1. **Multi-AZ RDS = HA, not performance** — read replicas for performance
2. **Aurora storage is shared** — adding a replica doesn't double storage costs
3. **DAX doesn't help for strongly consistent reads** — use ElastiCache Redis instead
4. **DynamoDB LSI must be created at table creation** — GSI can be added later
5. **ElastiCache Redis = persistence** — Memcached = no persistence
6. **RDS Proxy required for Lambda** — Lambda creates many short-lived connections
7. **Redshift is single AZ** by default (RA3 supports Multi-AZ)
8. **DynamoDB Global Tables = multi-active** — Aurora Global = 1 writer, multiple readers
