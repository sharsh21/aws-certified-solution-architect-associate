# AWS Solution Architect Associate (SAA-C03) — 7-Day Exam Guide

> **Exam:** SAA-C03 | **Duration:** 130 min | **Questions:** 65 | **Pass Score:** 720/1000
> **Format:** Multiple choice + Multiple response | **Cost:** $150 USD

---

## Exam Domain Weightage

| Domain | Topic | Weight |
|--------|-------|--------|
| Domain 1 | Design Secure Architectures | **30%** |
| Domain 2 | Design Resilient Architectures | **26%** |
| Domain 3 | Design High-Performing Architectures | **24%** |
| Domain 4 | Design Cost-Optimized Architectures | **20%** |

---

## 7-Day Study Plan

| Day | Focus Area | Files |
|-----|-----------|-------|
| Day 1 | Compute (EC2, Lambda, ECS, EKS) | `01_COMPUTE.md` |
| Day 2 | Storage (S3, EBS, EFS, FSx) | `02_STORAGE.md` |
| Day 3 | Networking (VPC, Route53, CloudFront, ELB) | `03_NETWORKING.md` |
| Day 4 | Databases (RDS, Aurora, DynamoDB, ElastiCache) | `04_DATABASE.md` |
| Day 5 | Security (IAM, KMS, WAF, GuardDuty) + Monitoring | `05_SECURITY.md` + `06_MONITORING.md` |
| Day 6 | Integration + Analytics + Migration + AI/ML | `07_INTEGRATION.md` + `08_ANALYTICS.md` + `09_MIGRATION_DR.md` + `11_AI_ML_SERVICES.md` |
| Day 7 | Full Mock Test + Weak Area Review + Exam Tips | `10_EXAM_TIPS.md` |

---

## All Files in This Guide

- [01_COMPUTE.md](01_COMPUTE.md) — EC2, Lambda, ECS, EKS, Elastic Beanstalk, Batch
- [02_STORAGE.md](02_STORAGE.md) — S3, EBS, EFS, FSx, Storage Gateway, Snow Family
- [03_NETWORKING.md](03_NETWORKING.md) — VPC, Route 53, CloudFront, ELB, API Gateway, Direct Connect
- [04_DATABASE.md](04_DATABASE.md) — RDS, Aurora, DynamoDB, ElastiCache, Redshift, Neptune
- [05_SECURITY.md](05_SECURITY.md) — IAM, KMS, CloudHSM, WAF, Shield, GuardDuty, Macie, Cognito
- [06_MONITORING.md](06_MONITORING.md) — CloudWatch, CloudTrail, Config, Trusted Advisor, Systems Manager
- [07_INTEGRATION.md](07_INTEGRATION.md) — SQS, SNS, EventBridge, Step Functions, Kinesis
- [08_ANALYTICS.md](08_ANALYTICS.md) — Athena, Glue, EMR, OpenSearch, QuickSight, Redshift Spectrum
- [09_MIGRATION_DR.md](09_MIGRATION_DR.md) — DMS, Snow Family, Backup, DR Strategies, CloudFormation
- [10_EXAM_TIPS.md](10_EXAM_TIPS.md) — Exam strategy, tricky question patterns, last-minute tips
- [11_AI_ML_SERVICES.md](11_AI_ML_SERVICES.md) — SageMaker, Rekognition, Comprehend, Lex, Bedrock, Textract, Polly, Translate

---

## Practice Questions (117 Questions Total)

| File | Topics | Questions |
|------|--------|-----------|
| [PRACTICE_Q1_COMPUTE_STORAGE.md](PRACTICE_Q1_COMPUTE_STORAGE.md) | EC2, Lambda, ECS, ELB, S3, EBS, EFS, FSx, Snow | Q1–Q28 (28 Qs) |
| [PRACTICE_Q2_NETWORKING_DB.md](PRACTICE_Q2_NETWORKING_DB.md) | VPC, Route 53, CloudFront, API GW, RDS, Aurora, DynamoDB, ElastiCache | Q29–Q58 (30 Qs) |
| [PRACTICE_Q3_SECURITY_MONITORING.md](PRACTICE_Q3_SECURITY_MONITORING.md) | IAM, KMS, WAF, GuardDuty, Macie, Cognito, CloudTrail, Config, SSM | Q59–Q84 (26 Qs) |
| [PRACTICE_Q4_INTEGRATION_ANALYTICS_DR_AI.md](PRACTICE_Q4_INTEGRATION_ANALYTICS_DR_AI.md) | SQS, SNS, Kinesis, Step Functions, Athena, Glue, DR, Migration, AI/ML | Q85–Q117 (33 Qs) |

---

## High-Frequency Exam Topics (Must Know)

### Always On the Exam
- **EC2 instance types** — when to use compute/memory/storage optimized
- **S3 storage classes** — when to use each (cost vs access frequency)
- **VPC** — subnets, NACLs vs Security Groups, VPC peering, Transit Gateway
- **RDS Multi-AZ vs Read Replicas** — HA vs performance distinction
- **IAM** — roles, policies, boundaries, STS AssumeRole
- **Auto Scaling + ELB** — ALB vs NLB vs CLB
- **Route 53 routing policies** — failover, latency, geolocation, weighted
- **SQS vs SNS vs EventBridge** — decoupling patterns
- **DynamoDB** — partition key design, GSI/LSI, DAX, Streams
- **CloudFront** — origin types, caching, signed URLs/cookies

---

## Critical Architecture Patterns

### High Availability
- Multi-AZ RDS + ALB + Auto Scaling Group
- S3 Cross-Region Replication (CRR)
- Route 53 Health Checks + Failover routing

### Cost Optimization
- Reserved Instances / Savings Plans
- S3 Intelligent-Tiering
- Spot Instances for fault-tolerant workloads
- Right-sizing with Compute Optimizer

### Security
- VPC + Private Subnets + NAT Gateway
- KMS encryption at rest
- CloudTrail + GuardDuty + Security Hub
- NACLs (stateless) + Security Groups (stateful)

### Decoupling
- SQS for async, SNS for fan-out, EventBridge for event-driven
- Step Functions for orchestration
- API Gateway + Lambda (serverless)

---

## Exam Answer Strategy

1. **Eliminate wrong answers first** — look for "most cost-effective", "least operational overhead", "most resilient"
2. **Managed service > self-managed** (exam favors AWS-managed solutions)
3. **Serverless > server-based** when operational overhead is a criterion
4. **Multi-AZ = HA**, **Read Replica = Performance/Scale**
5. **Security Group = stateful (instance level)**, **NACL = stateless (subnet level)**
6. **S3 = object store**, **EBS = block store (single AZ)**, **EFS = shared file system (multi-AZ)**
