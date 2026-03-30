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

### Keyspaces (Apache Cassandra)
- Managed Cassandra-compatible service
- CQL (Cassandra Query Language)
- Use for: existing Cassandra workloads on AWS

### Timestream
- Fully managed **time-series database**
- IoT applications, operational metrics, telemetry
- Fast ingestion, SQL queries

### QLDB — Quantum Ledger Database
- Immutable, cryptographically verifiable transaction log
- Use for: financial transactions, supply chain, audit history
- **Not decentralized** (unlike blockchain) — owned by central authority

### Managed Blockchain
- Create and manage blockchain networks
- Hyperledger Fabric, Ethereum
- Decentralized — no central authority

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
