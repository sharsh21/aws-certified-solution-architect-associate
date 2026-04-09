# Exam Strategy & Last-Minute Tips — SAA-C03

---

## Exam Day Strategy

### Time Management
- **65 questions in 130 minutes** = 2 min/question
- Flag uncertain questions and revisit
- Aim to finish in 110 min, use last 20 min for review
- Don't spend > 3 min on any single question

### Answering Technique
1. **Read the last sentence first** — it has the actual question
2. **Identify the key constraint:** most cost-effective, least operational overhead, highest availability, most secure, minimum downtime
3. **Eliminate obviously wrong answers** (usually 2 can be eliminated quickly)
4. **Managed > self-managed** (AWS exam prefers managed services)
5. **Serverless > servers** when operational overhead is the criteria
6. If stuck: pick the more **resilient, secure, scalable** option

---

## Keyword → Service Mapping (CRITICAL)

### By Requirement

| Keyword in Question | Think |
|--------------------|----|
| "least operational overhead" | Serverless, managed services (Lambda, Fargate, Aurora Serverless) |
| "most cost-effective" | Spot, Reserved, S3 Intelligent-Tiering, Glacier, serverless |
| "high availability" | Multi-AZ, ASG, ELB, Route 53 failover |
| "fault tolerant" | Spot + checkpointing, Multi-AZ, Cross-region replication |
| "scalability" | ASG, DynamoDB, Lambda, S3 |
| "minimum downtime migration" | DMS + CDC, Blue/Green, Canary deployment |
| "decouple components" | SQS, SNS, EventBridge |
| "real-time processing" | Kinesis Data Streams, Lambda |
| "near real-time batch" | Kinesis Firehose |
| "compliance, audit trail" | CloudTrail, Config, QLDB |
| "sensitive data discovery" | Macie |
| "threat detection" | GuardDuty |
| "vulnerability scanning" | Inspector |
| "centralized secret rotation" | Secrets Manager |
| "config values, no rotation" | SSM Parameter Store |
| "access S3 without internet" | VPC Gateway Endpoint |
| "private connectivity to AWS services" | VPC Interface Endpoint (PrivateLink) |
| "static IP for load balancer" | NLB |
| "WebSocket / real-time bidirectional" | API Gateway WebSocket, AppSync |
| "GraphQL" | AppSync |
| "global, low-latency reads" | Aurora Global Database, CloudFront, DynamoDB Global Tables |
| "offline sync mobile" | AppSync + DynamoDB |
| "OLAP / data warehouse" | Redshift |
| "ad-hoc SQL on S3" | Athena |
| "ETL pipeline" | Glue |
| "graph database" | Neptune |
| "time series" | Timestream |
| "immutable audit ledger" | QLDB |
| "existing Kafka" | MSK |
| "existing ActiveMQ/RabbitMQ" | Amazon MQ |
| "existing Cassandra" | Keyspaces |
| "existing MongoDB" | DocumentDB |
| "HPC / tightly coupled" | EC2 Cluster Placement Group, FSx for Lustre, EFA |
| "on-premises to AWS ongoing" | Storage Gateway, Direct Connect |
| "on-premises to AWS one-time" | DataSync, Snow Family |
| "hybrid file sharing (Windows)" | FSx File Gateway / FSx for Windows |
| "hybrid file sharing (Linux)" | EFS / S3 File Gateway |
| "large data transfer offline" | Snow Family |
| "DR lowest cost" | Backup and Restore |
| "DR fastest recovery" | Multi-Site Active/Active |
| "blue/green deployment" | Elastic Beanstalk, CodeDeploy, Route 53 weighted |
| "in-memory cache, simple" | ElastiCache Memcached |
| "leaderboard, pub/sub, persistence" | ElastiCache Redis |
| "EC2 access without bastion" | Systems Manager Session Manager |
| "certificate for CloudFront" | ACM in us-east-1 |
| "cross-account event routing" | EventBridge |
| "fan-out" | SNS → multiple SQS |
| "exactly-once ordering" | SQS FIFO |
| "replay messages" | Kinesis Data Streams |

---

## Must-Know Numbers

### Limits
| Service | Limit |
|---------|-------|
| S3 max object size | 5 TB |
| S3 multipart required above | 5 GB |
| DynamoDB max item size | 400 KB |
| Lambda max execution time | 15 minutes |
| Lambda /tmp storage | 10 GB |
| Lambda default concurrency | 1,000 |
| SQS message max size | 256 KB |
| SQS max retention | 14 days |
| SQS visibility timeout default | 30 seconds |
| Kinesis shard ingestion | 1 MB/s |
| Kinesis shard egress | 2 MB/s |
| Kinesis default retention | 24 hours (up to 365 days) |
| RDS Read Replicas (MySQL/PG) | 5 |
| Aurora Read Replicas | 15 |
| Aurora max storage | 128 TB |
| VPC default limit | 5 per region |
| EIP default limit | 5 per region |
| ELB SG HTTPS redirect | 443 |
| CloudFront default TTL | 86400 sec (24 hours) |
| Route 53 health check interval | 10 or 30 seconds |

### Storage Class Minimum Days
| Class | Min Days |
|-------|---------|
| Standard-IA | 30 |
| One Zone-IA | 30 |
| Glacier Instant | 90 |
| Glacier Flexible | 90 |
| Glacier Deep Archive | 180 |

---

## High-Frequency Architecture Scenarios

### Scenario 1: Highly Available 3-Tier App
**Answer pattern:** ALB + EC2 Auto Scaling Group (multi-AZ) + RDS Multi-AZ + NAT Gateway (each AZ)

### Scenario 2: Serverless Web App
**Answer pattern:** Route 53 → CloudFront → API Gateway → Lambda → DynamoDB

### Scenario 3: Cost-Optimize EC2
**Answer pattern:** On-Demand (baseline) + Spot (flexible scale) + Reserved (base load) + Savings Plans

### Scenario 4: Secure S3 Access from CloudFront
**Answer pattern:** CloudFront + OAC (Origin Access Control) + S3 Bucket Policy → block direct S3 access

### Scenario 5: Private EC2 Access Without Bastion
**Answer pattern:** SSM Session Manager (IAM role with AmazonSSMManagedInstanceCore policy)

### Scenario 6: Multi-Region DR
**Answer pattern:** Aurora Global Database + S3 CRR + Route 53 failover routing

### Scenario 7: Decouple Microservices
**Answer pattern:** SQS between services (sync → async) or SNS fan-out → SQS per service

### Scenario 8: Migrate Large Data to S3
- < 1 week with internet: DataSync or S3 Transfer Acceleration
- > 1 week / bandwidth constrained: Snow Family

### Scenario 9: Centralize Logs from Multiple Accounts
**Answer pattern:** CloudWatch Logs → Kinesis Data Firehose → S3 (central logging account)

### Scenario 10: Compliance — Prevent Deletion of Logs
**Answer pattern:** S3 Object Lock (Compliance mode) + S3 Versioning + MFA Delete

---

## Common Traps Summary

### Multi-AZ vs Read Replica
- **Multi-AZ = HA (failover, no downtime)** — NOT for reads, NOT for performance
- **Read Replica = scale reads** — NOT for HA, NOT automatic failover

### Security Groups vs NACLs
- **SG = stateful (instance level)** — return traffic auto allowed
- **NACL = stateless (subnet level)** — must allow both directions, use ephemeral ports outbound

### S3 vs EBS vs EFS
- **S3 = object** (no mount, web API, unlimited)
- **EBS = block** (mount to 1 EC2, same AZ, persistent)
- **EFS = file NFS** (mount to many EC2, multi-AZ, Linux only)

### ALB vs NLB
- **ALB = L7, HTTP/HTTPS, path routing, WAF** — no static IP
- **NLB = L4, TCP/UDP, static IP, ultra-low latency** — no HTTP routing

### CloudFront vs Global Accelerator
- **CloudFront = content caching (CDN)**, reduces latency for static content
- **Global Accelerator = network routing**, static anycast IPs, TCP/UDP acceleration

### SNS vs SQS vs EventBridge
- **SNS = push fan-out** (deliver to all subscribers immediately)
- **SQS = queue decoupling** (messages wait, one consumer)
- **EventBridge = event routing** (rule-based, AWS service events)

---

## Quick Service Purpose Reference

| Service | One-Line Purpose |
|---------|-----------------|
| CloudFront | CDN / cache content at edge |
| Global Accelerator | Route TCP/UDP to nearest endpoint, static IPs |
| Route 53 | DNS + health checks + routing policies |
| VPC | Isolated private network |
| Direct Connect | Dedicated private network to AWS |
| Transit Gateway | Hub-and-spoke multi-VPC networking |
| WAF | Block bad HTTP requests (SQL injection, XSS) |
| Shield | DDoS protection |
| GuardDuty | Threat detection (ML-based) |
| Macie | Sensitive data discovery in S3 |
| Inspector | CVE vulnerability scanning |
| Config | Resource configuration compliance |
| CloudTrail | API call audit logging |
| CloudWatch | Metrics, logs, alarms |
| Secrets Manager | Store and rotate secrets |
| SSM Parameter Store | Store config values and secrets |
| KMS | Key management, encryption |
| CloudHSM | Dedicated HSM hardware |
| Cognito | User auth + AWS credentials for web/mobile |
| STS | Temporary AWS credentials |
| Organizations | Multi-account management |
| Control Tower | Landing zone automation |
| Glue | Serverless ETL |
| Athena | Serverless SQL on S3 |
| EMR | Managed Hadoop/Spark |
| Kinesis Streams | Real-time data streaming |
| Kinesis Firehose | Near-real-time delivery to destinations |
| MSK | Managed Kafka |
| Amazon MQ | Managed ActiveMQ/RabbitMQ |
| SQS | Message queue |
| SNS | Pub/Sub notifications |
| EventBridge | Event bus / event routing |
| Step Functions | Workflow orchestration |
| DMS | Database migration |
| DataSync | File/data migration |
| Snow Family | Offline data transfer |
| Storage Gateway | Hybrid on-prem to cloud storage |
| AWS Backup | Centralized backup management |
| Outposts | AWS infra on-premises |
| Local Zones | AWS in metro areas |
| Wavelength | AWS in 5G carrier networks |

---

## Day-Before Checklist

- [ ] Review all storage class minimum durations
- [ ] Review Multi-AZ vs Read Replica distinction
- [ ] Review SG vs NACL distinction
- [ ] Review all Route 53 routing policies
- [ ] Review VPC Gateway vs Interface Endpoints
- [ ] Review IAM policy evaluation order
- [ ] Review DR strategy RTO/RPO comparison
- [ ] Know when to use Kinesis vs SQS
- [ ] Review Aurora Global Database vs DynamoDB Global Tables
- [ ] Confirm ELB type selection criteria

---

## Exam Registration Reminders
- Bring government-issued photo ID
- 1 additional ID acceptable
- Pearson VUE: online or test center
- No paper, no phone during exam
- 30 minutes additional time available (ESA accommodation)
- Results: immediate preliminary, official within 5 business days
