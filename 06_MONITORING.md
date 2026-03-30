# Monitoring & Management Services — SAA-C03 Exam Guide

---

## CloudWatch

### Core Components

#### Metrics
- Time-series data for AWS services and custom apps
- **Default metrics:** CPU, NetworkIn/Out, DiskRead/Write, StatusCheck
- **Custom metrics:** Push via PutMetricData API (application-level metrics)
- **Namespaces:** Logical grouping of metrics (e.g., AWS/EC2, AWS/RDS)
- **Dimensions:** Name-value pairs to filter metrics (InstanceId, AutoScalingGroupName)
- **Resolution:** Standard (1 min) or High Resolution (1 second, extra cost)
- **Retention:** 1 sec = 3 hours, 60 sec = 15 days, 5 min = 63 days, 1 hour = 15 months

#### Key Missing EC2 Metrics (CRITICAL)
- **NOT available by default:** Memory utilization, Disk space used, Swap usage
- Must use **CloudWatch Agent** to collect these

#### CloudWatch Agent
- Runs on EC2 (and on-premises)
- Collects: system metrics (memory, disk), logs
- Requires: IAM role with `CloudWatchAgentServerPolicy`
- Config stored in SSM Parameter Store

#### CloudWatch Alarms
- Watch a single metric, trigger actions
- **States:** OK, ALARM, INSUFFICIENT_DATA
- **Actions:** EC2 actions (reboot, stop, recover), Auto Scaling, SNS notification
- **Composite Alarms:** Combine multiple alarms (AND/OR logic)
- Alarm on Math Expression across metrics

#### CloudWatch Logs
- Collect, monitor, store logs
- **Log Groups:** Logical container for log streams (e.g., /aws/lambda/my-function)
- **Log Streams:** Sequence of events from a single source
- **Retention Policy:** 1 day to 10 years (default: never expire)
- **Export to S3:** Batch export (up to 12 hours delay)
- **Logs Insights:** Interactive query and visualization (SQL-like)
- **Log Subscriptions:** Real-time stream to Lambda, Kinesis Data Streams, Kinesis Firehose
- **Metric Filters:** Extract metric data from log patterns

#### CloudWatch Logs Destinations (Cross-account/region)
- Kinesis Data Streams → Lambda / Kinesis Firehose
- Cross-account logging: subscription filters + Kinesis

#### CloudWatch Container Insights
- Metrics and logs for ECS, EKS, Kubernetes on EC2
- Fargate-supported

#### CloudWatch Lambda Insights
- System-level metrics for Lambda functions

#### CloudWatch Application Insights
- Automated monitoring for .NET and SQL Server apps

---

## CloudTrail

- **Audit API calls** to AWS (who did what, when, from where)
- Records: management events (default) + data events (optional) + Insights events
- **Management Events:** Control plane operations (CreateBucket, RunInstances, etc.)
- **Data Events:** S3 object operations, Lambda invocations, DynamoDB item-level (not enabled by default — costs extra)
- **CloudTrail Insights:** Detect unusual API activity (enabled separately)

### CloudTrail Storage
- Delivered to S3 within **15 minutes**
- 90-day history in Event History (free)
- Create Trail → store indefinitely in S3
- Encrypt with KMS

### Cross-Account/Multi-Region
- Multi-region trail: single trail covering all regions
- Organization Trail: apply to all accounts in AWS Organizations
- S3 bucket can be in different account (central logging account)

> **Exam Tip:** CloudTrail = API audit trail. CloudWatch = performance metrics and operational monitoring. Different purposes.

---

## AWS Config

- **Configuration compliance** — track configuration changes, evaluate against rules
- Records resource configuration history
- **Config Rules:** Evaluate compliance of resources
  - AWS Managed Rules: 150+ predefined rules
  - Custom Rules: Lambda functions
- **Remediation:** Auto-remediate non-compliant resources (SSM Automation)
- **Conformance Packs:** Bundle of Config rules + remediation
- Multi-account, multi-region aggregation

### Common Config Rules
- `restricted-ssh` — no inbound SSH from 0.0.0.0/0
- `s3-bucket-public-read-prohibited` — S3 not publicly readable
- `encrypted-volumes` — EBS volumes encrypted
- `rds-storage-encrypted` — RDS encrypted at rest

> **Exam Tip:** Config = compliance and drift detection. NOT a real-time blocking service — detects and can remediate, but doesn't prevent.

---

## AWS Organizations

### Key Concepts
- **Management Account (Root/Master):** Central account that creates the Org
- **Member Accounts:** Managed accounts
- **OUs (Organizational Units):** Hierarchical groupings
- **SCPs (Service Control Policies):** Permission guardrails on accounts/OUs

### SCP Behavior
- **Don't grant permissions** — they define maximum permissions
- Apply to all users/roles in affected account (including root user of member accounts)
- **NOT applied to management account**
- Use to restrict regions, services, or specific actions

### Consolidated Billing
- Single payment method for all accounts
- Combine usage for volume discounts (S3, EC2 Reserved Instances)
- Reserved Instance sharing across accounts
- Savings Plans sharing

### Other Features
- **Tag Policies:** Enforce tag consistency
- **AI Services Opt-out Policies:** Opt out of AWS AI services using your data
- **Backup Policies:** Centralized backup governance

---

## AWS Control Tower

- Automates multi-account AWS environment setup (landing zone)
- Pre-configured guardrails (preventive with SCPs, detective with Config)
- **Account Factory:** Provision new accounts with approved baseline
- **Dashboard:** Compliance status across accounts
- Built on top of AWS Organizations

---

## Trusted Advisor

- Automated recommendations across 5 categories:
  1. **Cost Optimization** (unused resources, Reserved Instance recommendations)
  2. **Performance** (high utilization, over-provisioned)
  3. **Security** (open ports, MFA on root, public S3 buckets)
  4. **Fault Tolerance** (Multi-AZ, backups)
  5. **Service Limits** (approaching limits)

| Plan | Checks Available |
|------|----------------|
| Basic/Developer | 7 core checks |
| Business/Enterprise | All checks + AWS Support API |

---

## AWS Well-Architected Tool

- Review workloads against Well-Architected Framework
- 6 Pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability
- Generates improvement plan

---

## AWS Compute Optimizer

- ML-based recommendations for right-sizing
- Covers: EC2, EC2 Auto Scaling, Lambda, EBS, ECS on Fargate
- Requires CloudWatch metrics (14+ days)
- Identifies over/under-provisioned resources

---

## AWS Systems Manager (Key Features)

- **Parameter Store:** Config values and secrets (with/without KMS)
- **Session Manager:** Secure shell access without SSH/bastion (audit-logged)
- **Patch Manager:** Automated patching with maintenance windows
- **Run Command:** Execute commands on managed instances remotely
- **Automation:** Runbook-based automation (create AMIs, patch, etc.)
- **Inventory:** Hardware/software inventory from EC2 instances
- **OpsCenter:** View and investigate operational issues
- **Explorer:** Operational dashboard across regions/accounts

> **Exam Tip:** SSM Session Manager is the preferred way to access EC2 in private subnets — no bastion, no port 22, full audit trail.

---

## AWS Health Dashboard

- **Service Health Dashboard:** Status of all AWS services globally (public)
- **Personal Health Dashboard (AWS Health):** Events affecting YOUR resources specifically
- Proactive notifications for maintenance, incidents

---

## Monitoring Architecture Patterns

### Centralized Logging
```
All Accounts → CloudTrail → S3 (central logging account)
EC2/Lambda → CloudWatch Logs → Logs Subscription → Kinesis → S3/OpenSearch
```

### Security Monitoring
```
GuardDuty + Macie + Inspector → Security Hub → EventBridge → Lambda (auto-remediation)
```

### Cost Monitoring
```
Cost Explorer + Budgets → SNS Alert → email/Slack notification
```

---

## Key Exam Traps

1. **EC2 memory/disk metrics NOT in CloudWatch by default** — need CloudWatch Agent
2. **CloudTrail 90-day history** is free, but you need a Trail for longer retention
3. **Config doesn't prevent changes** — it detects and can remediate, not block
4. **SCP doesn't apply to management account** — master account is always exempt
5. **CloudWatch Logs export to S3** has up to 12-hour delay; use subscription filters for real-time
6. **Trusted Advisor full checks** require Business/Enterprise support plan
7. **CloudTrail data events are not enabled by default** — management events are default
