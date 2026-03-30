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
