# Analytics & Big Data Services — SAA-C03 Exam Guide

---

## Athena

- **Serverless SQL** query engine for data in S3
- No infrastructure, no ETL required
- Supports: CSV, JSON, Parquet, ORC, Avro
- **Pay per query:** $5 per TB scanned
- **Columnar formats (Parquet/ORC)** reduce cost 30-90% and improve performance
- **Partitioning:** Organize data by date/region to reduce scan cost
- Works with AWS Glue Data Catalog for schema metadata
- Use for: log analysis, ad-hoc queries, quick insights on S3 data

### Athena Federated Query
- Query data beyond S3 (RDS, DynamoDB, CloudWatch, on-premises)
- Uses Lambda connectors

### Athena Performance Tips
- Use columnar format (Parquet, ORC)
- Use compression (Snappy, GZIP)
- Partition data by common query filters
- Use Athena CTAS (Create Table As Select) to convert formats

### Real-World Use Cases & When to Use Athena

> **The problem it solves:** Run ad-hoc SQL queries directly on S3 data without loading it into a database — no ETL, no infrastructure, pay only for data scanned.

**Choose Athena when:** Data is already in S3, you need quick analysis without an ETL pipeline, or analysts want SQL access to a data lake.

**Real-World Scenarios:**
| Business Problem | Athena Solution |
|-----------------|----------------|
| Security team needs to analyze 6 months of CloudTrail logs (stored in S3) for suspicious activity | Athena query on CloudTrail S3 bucket — `SELECT * FROM cloudtrail WHERE eventName = 'ConsoleLogin' AND errorCode IS NOT NULL` |
| Finance team wants monthly revenue reports from 5TB of transaction CSV files in S3 | Athena — SQL query, results in seconds. Convert to Parquet first → 90% cost reduction |
| Business wants to query DynamoDB and S3 together in one SQL | Athena **Federated Query** — Lambda connector bridges DynamoDB + S3 data lake |
| Dev team needs to quickly validate data quality of a new S3 data dump | Athena ad-hoc queries — no infrastructure provisioning, just point at S3 and query |

**Cost optimization rule for the exam:** CSV → $5/TB scanned. Same data in Parquet + Snappy compression → $0.50/TB (10x cheaper). Always recommend converting to Parquet + partitioning.

---

## AWS Glue

- Fully managed **ETL (Extract, Transform, Load)** service
- **Serverless** — no cluster management
- Key components:
  - **Glue Crawlers:** Scan data sources, infer schemas, populate Glue Data Catalog
  - **Glue Data Catalog:** Central metadata repository (tables, schemas, partitions)
  - **Glue ETL Jobs:** Python/Scala Spark jobs for transformation
  - **Glue Studio:** Visual ETL builder
  - **Glue DataBrew:** Visual data preparation (no code)
  - **Glue Elastic Views:** Materialized views across data stores

### Glue Data Catalog
- Central schema store for Athena, Redshift Spectrum, EMR, Lake Formation
- One per region per account
- Integrates with Apache Hive Metastore

### Glue Streaming ETL
- Reads from Kinesis Data Streams or Kafka
- Real-time ETL processing

### Real-World Use Cases & When to Use AWS Glue

> **The problem it solves:** Serverless ETL — automatically discover schemas (Crawlers), transform data with Python/Spark, and load it into analytics targets — without managing Spark clusters.

**Real-World Scenarios:**
| Business Problem | Glue Solution |
|-----------------|--------------|
| Data lake ingestion: raw JSON logs in S3 need to be converted to Parquet for Athena | Glue ETL Job — reads JSON, applies schema, writes Parquet partitioned by date → 90% cost savings on Athena |
| Analytics team needs a catalog of all data across 50 S3 buckets + RDS | Glue **Crawler** — scans all sources, auto-generates tables in Glue Data Catalog |
| Marketing wants to join S3 clickstream data with RDS customer data | Glue ETL — reads from S3 + RDS, joins, writes enriched dataset to Redshift |
| Business analyst wants to clean data without writing code | **Glue DataBrew** — visual point-and-click data transformations, 250+ built-in transformations |
| Streaming ETL: Kinesis events need real-time enrichment before loading to S3 | **Glue Streaming ETL** — reads KDS, transforms, writes to S3/Redshift in near real-time |

**Glue Data Catalog is the metadata layer for the entire analytics ecosystem:** Athena, Redshift Spectrum, and EMR all use the same Glue Data Catalog — define the schema once, use everywhere.

---

## EMR — Elastic MapReduce

- Managed **Hadoop/Spark** cluster on EC2
- Supports: Spark, Hive, HBase, Flink, Presto, Pig
- Use for: large-scale data processing, ML, ETL
- **Node Types:**
  - **Master:** Coordinates cluster
  - **Core:** Run tasks + store data (HDFS)
  - **Task:** Run tasks only (no storage, can use Spot)
- EMR on EKS: run Spark on EKS
- EMR Serverless: no cluster management

### EMR vs Glue vs Athena
| | Athena | Glue ETL | EMR |
|-|--------|---------|-----|
| Model | Serverless SQL | Serverless ETL | Managed Hadoop/Spark |
| Coding | SQL | Python/Scala | Python/Scala/Java |
| Control | None | Low | High |
| Use Case | Ad-hoc S3 queries | Data transformation | Custom big data processing |

### Real-World Use Cases & When to Use EMR

> **The problem it solves:** Run large-scale distributed data processing (Hadoop, Spark, Hive, Flink) on managed EC2 clusters — full control over the framework, spot instance optimization, and custom configurations that serverless Glue can't provide.

**Choose EMR when:** You need Spark/Hadoop frameworks with full control, custom configurations, or very large-scale processing that needs to be cost-optimized with Spot instances.

**Real-World Scenarios:**
| Business Problem | EMR Solution |
|-----------------|-------------|
| Data team processes 10PB of clickstream data daily — Glue too slow | EMR Spark cluster — 100 core nodes (Spot), custom Spark config, full control |
| Biotech runs 10,000-node Hadoop job for genomics analysis | EMR with task nodes on Spot — massive parallel processing, 80% cost savings |
| Log processing: 500GB/hour Apache logs need custom Spark streaming logic | EMR Flink — stateful stream processing, exactly-once semantics, custom windows |
| Data science team uses PySpark notebooks for exploratory analysis | **EMR Studio** — managed Jupyter notebooks connected to EMR cluster |
| ML team needs distributed feature engineering on 100TB dataset | EMR Spark with SageMaker integration — distributed feature processing → S3 → SageMaker training |

**EMR cost strategy for the exam:** Master node = On-Demand (long-running). Core nodes = On-Demand (HDFS data). Task nodes = **Spot** (compute only, no HDFS — safe to interrupt).

---

## OpenSearch (formerly Elasticsearch)

- Managed **search and analytics** engine
- Use for: full-text search, log analytics, real-time dashboards
- **OpenSearch Dashboards (Kibana):** Visualization
- Data sources: Kinesis, DynamoDB Streams, CloudWatch Logs, S3, Lambda
- Multi-AZ deployment: 3 AZ recommended
- **UltraWarm:** Low-cost warm tier (S3 backed)
- **Cold Storage:** Even cheaper for archived indices

> **Exam Tip:** OpenSearch = search + log analytics. When you see "search" or "log analysis with dashboards" → OpenSearch.

### Real-World Use Cases & When to Use OpenSearch

> **The problem it solves:** Full-text search and log analytics at scale — search across billions of documents in milliseconds, and visualize log data with dashboards (formerly Kibana, now OpenSearch Dashboards).

**Real-World Scenarios:**
| Business Problem | OpenSearch Solution |
|-----------------|---------------------|
| E-commerce site: users type "blue running shoes size 10" — needs fuzzy search | OpenSearch — full-text search with fuzzy matching, faceted filtering, relevance scoring |
| DevOps team needs to search application logs across 100 EC2 instances | CloudWatch Logs → Kinesis Firehose → OpenSearch → OpenSearch Dashboards (log visualization) |
| Security team needs SIEM — correlate events across VPC Flow Logs + CloudTrail | Kinesis → OpenSearch — index all logs, alert on anomalies, Dashboards for investigation |
| Product catalog: 5M items with complex attribute search (brand, color, price range) | OpenSearch — structured + full-text hybrid search with filters |
| Media platform: archive index 2TB old indices but keep them searchable cheaply | OpenSearch **UltraWarm** — S3-backed warm tier, 90% cheaper than hot storage |

**The keyword triggers:** "search", "full-text search", "log analytics with dashboards", "Elasticsearch equivalent" → OpenSearch.

---

## Kinesis (Recap in Analytics Context)

### Kinesis Data Analytics
- Run SQL or **Apache Flink** on streaming data
- Input: Kinesis Data Streams or Kinesis Firehose
- Output: S3, Redshift, OpenSearch, Lambda
- **Managed Apache Flink:** Complex streaming ETL, stateful processing, ML

### Real-Time Analytics Architecture
```
IoT/Apps → Kinesis Data Streams → Kinesis Data Analytics (Flink) → OpenSearch/S3
                               → Lambda (real-time alerts)
                               → Kinesis Firehose → Redshift/S3
```

---

## QuickSight

- Fully managed **BI (Business Intelligence)** and visualization tool
- Serverless, scales automatically
- Sources: S3, Athena, RDS, Aurora, Redshift, OpenSearch, SalesForce, etc.
- **SPICE:** In-memory engine for fast queries (Superfast, Parallel, In-memory Calculation Engine)
- **ML Insights:** Anomaly detection, forecasting, natural language narratives
- **Q:** Natural language queries
- Pricing: per-user or capacity pricing

---

## AWS Data Pipeline (Legacy)

- Orchestrate data movement/transformation across services
- More complex than Glue — largely replaced by Glue/Step Functions
- Still appears on exam occasionally

---

## Lake Formation

- Build, secure, and manage **data lakes** on S3
- Fine-grained access control (column, row, cell level)
- Centralized permissions on top of S3 + Glue Data Catalog
- Cross-account data sharing
- Simplifies data lake setup (uses Glue, S3, Athena under the hood)

### Real-World Use Cases & When to Use Lake Formation

> **The problem it solves:** Govern a data lake with fine-grained security without managing S3 bucket policies per dataset — centralize who can see which tables, columns, and rows across Athena, Redshift Spectrum, and EMR.

**Real-World Scenarios:**
| Business Problem | Lake Formation Solution |
|-----------------|------------------------|
| Data lake has PII columns — HR can see salary data, developers cannot | **Column-level security** — HR role sees salary column, dev role policy excludes it |
| EU data must only be visible to EU-based analysts (GDPR) | **Row-level filtering** — filter rows where `country = 'EU'` based on user's IAM tag |
| Sharing anonymized dataset with external partner's AWS account | Lake Formation **cross-account data sharing** — share Glue catalog tables without copying data |
| Setting up a new data lake takes 3 months — need to speed up | Lake Formation blueprints — automates S3 + Glue + Athena + permissions in hours |

---

## MSK — Managed Streaming for Apache Kafka

- Fully managed **Apache Kafka**
- Alternative to Kinesis for Kafka workloads
- Standard Kafka APIs, tools, and integrations
- MSK Serverless: auto-scale capacity
- Data retention: configurable

### Kinesis vs MSK
| | Kinesis Data Streams | MSK |
|-|---------------------|-----|
| Managed | Yes (shard-based) | Yes (Kafka-native) |
| Protocol | Proprietary | Kafka standard |
| Scaling | Shard split/merge | Add/remove brokers |
| Use Case | AWS-native streaming | Kafka migrations, Kafka tooling |

### Real-World Use Cases & When to Use MSK

> **The problem it solves:** Run fully managed Apache Kafka on AWS — same Kafka APIs, same tools (Kafka Connect, MirrorMaker, Flink), no cluster management.

**Choose MSK when:** Migrating existing Kafka workloads, using Kafka ecosystem tooling (Kafka Connect, ksqlDB), or need Kafka-specific features (topics, consumer groups, compaction).

**Real-World Scenarios:**
| Business Problem | MSK Solution |
|-----------------|-------------|
| Ride-sharing company runs Kafka on-premises for real-time location tracking — migrating to AWS | MSK — same Kafka producer/consumer code, same topics, zero app changes |
| Event streaming platform uses Kafka Connect to stream from 30 source systems | MSK with Kafka Connect (MSK Connect) — managed connectors, auto-scaling |
| Compliance: Kafka messages must be retained for 90 days | MSK — configurable retention per topic (unlike Kinesis max 365 days, MSK is configurable) |
| Data team uses ksqlDB for real-time SQL on Kafka streams | MSK + ksqlDB — Kafka-native stream processing with SQL |

**MSK vs Kinesis for the exam:** "Kafka migration" or "Kafka protocol" → MSK. "AWS-native streaming, no Kafka expertise" → Kinesis. **MSK Serverless** = auto-scale capacity without broker management.

---

## Analytics Architecture Patterns

### Modern Data Lake Architecture
```
Raw Data → S3 (Data Lake)
         → Glue Crawler → Glue Data Catalog
         → Athena (SQL queries)
         → QuickSight (dashboards)
         → EMR (custom processing)
```

### Real-Time + Batch Pipeline
```
                    → Kinesis Data Analytics → S3/OpenSearch (real-time)
Data Source → KDS →
                    → Kinesis Firehose → S3 → Glue ETL → Redshift (batch BI)
```

### Log Analytics Stack
```
EC2/Lambda → CloudWatch Logs → Kinesis Firehose → OpenSearch → Dashboards
                             → S3 → Athena (ad-hoc)
```

---

## Key Exam Traps

1. **Athena charges per TB scanned** — use Parquet/ORC + partitioning to reduce cost
2. **Glue Data Catalog ≠ Glue ETL** — catalog is just metadata, ETL is the transform job
3. **Kinesis Firehose is near real-time** (60s minimum) — not sub-second
4. **OpenSearch ≠ database** — it's for search and analytics, not transactional storage
5. **QuickSight SPICE** — data cached in-memory; doesn't query source on every visualization
6. **EMR Task nodes = no HDFS storage** — can run on Spot safely (data on Core nodes)
7. **Athena Federated Query** requires Lambda connectors and results stored in S3
