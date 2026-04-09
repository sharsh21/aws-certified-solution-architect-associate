# Migration, Disaster Recovery & Infrastructure — SAA-C03 Exam Guide

---

## Disaster Recovery Strategies (CRITICAL — MEMORIZE)

### DR Strategy Comparison

| Strategy | RTO | RPO | Cost | Description |
|---------|-----|-----|------|-------------|
| **Backup & Restore** | Hours | Hours | Lowest | Backup to S3/Glacier, restore on disaster |
| **Pilot Light** | ~10 min | Minutes | Low | Core services running (DB), scale up on disaster |
| **Warm Standby** | Minutes | Seconds | Medium | Scaled-down version always running |
| **Multi-Site Active/Active** | Real-time | Near-zero | Highest | Full capacity in multiple regions |

> **Exam Tip:**
> - Lower RTO/RPO = higher cost
> - "Lowest cost DR" → Backup & Restore
> - "Fastest recovery, any cost" → Multi-Site Active/Active
> - Pilot Light: DB is running, EC2 is stopped → start EC2 on disaster
> - Warm Standby: Both DB and EC2 running at minimum, scale up on disaster

### RTO vs RPO
- **RTO (Recovery Time Objective):** How long to recover = downtime tolerance
- **RPO (Recovery Point Objective):** How much data loss = backup frequency

### Real-World DR Strategy Selection

> **The core principle:** Match DR strategy to business requirements. Every reduction in RTO/RPO adds cost.

**DR Strategy Selection by Business Scenario:**
| Company Type | RTO/RPO Tolerance | DR Strategy |
|-------------|------------------|-------------|
| Personal blog / non-critical internal tool | Hours/Hours | **Backup & Restore** — nightly S3 backup, restore from snapshot |
| SMB e-commerce — can tolerate 30 min downtime | ~10 min / Minutes | **Pilot Light** — Aurora Multi-AZ running, EC2 AMI ready, start fleet on disaster |
| SaaS product — SLA says 99.9% uptime (8.7 hrs/year) | Minutes / Seconds | **Warm Standby** — scaled-down copy always running in DR region, scale up on failover |
| Stock exchange, banking core system — zero tolerance | Near-zero / Near-zero | **Multi-Site Active/Active** — Route 53 latency routing across 2 regions, both serve live traffic |
| Healthcare system with patient records — 1-hour RPO max | 15 min / 1 hour | **Pilot Light** with hourly RDS snapshots + Aurora cross-region replica |

**Architect's mental model for the exam:**
- Backup & Restore = "we save files, restore if disaster"
- Pilot Light = "engine is running (DB), but car is parked (EC2 stopped)"
- Warm Standby = "a smaller car is driving slowly, we floor it when disaster strikes"
- Active/Active = "two cars driving the same route simultaneously"

---

## AWS Backup

- Centralized backup management across AWS services
- Supports: EC2, EBS, RDS, Aurora, DynamoDB, EFS, FSx, S3, Storage Gateway
- **Backup Plans:** Schedule, retention, lifecycle rules
- **Backup Vault:** Storage for backups (KMS encrypted)
- **Cross-region and cross-account backup**
- **Backup Vault Lock (WORM):** Prevent backup deletion even by root
- **AWS Backup Audit Manager:** Compliance reports

---

## AWS DMS — Database Migration Service

- Migrate databases with **minimal downtime**
- Source DB stays online during migration
- Supports: Oracle, SQL Server, MySQL, PostgreSQL, MongoDB, S3, Redshift, DynamoDB

### Migration Types
| Type | Description |
|------|-------------|
| **Full Load** | Migrate entire dataset at once |
| **Full Load + CDC** | Migrate data + ongoing changes (continuous replication) |
| **CDC Only** | Only capture and replicate ongoing changes |

### Schema Conversion Tool (SCT)
- Required for **heterogeneous migrations** (Oracle → Aurora, SQL Server → PostgreSQL)
- Converts DDL, stored procedures, functions
- **Not needed** for homogeneous (MySQL → RDS MySQL, Oracle → RDS Oracle)

### DMS Replication Instance
- EC2 instance that runs replication tasks
- Located in VPC
- Can migrate from on-premises (with Direct Connect or VPN)

### Real-World Use Cases & When to Use DMS

> **The problem it solves:** Migrate databases to AWS with minimal downtime — source database stays online and serving traffic during the entire migration, then you do a brief cutover.

**Real-World Scenarios:**
| Business Problem | DMS Solution |
|-----------------|-------------|
| Migrate Oracle DB (on-premises, 500GB) to Aurora PostgreSQL — can't have > 1 hr downtime | DMS + SCT (schema conversion) — full load + CDC keeps source in sync, cut over in minutes |
| Migrate MySQL on EC2 to RDS MySQL (same engine) — no schema changes | DMS homogeneous migration (no SCT) — full load + CDC, near-zero downtime |
| Ongoing replication from on-premises SQL Server → Redshift for analytics | DMS CDC only — continuously replicate changes for real-time analytics warehouse |
| Data warehouse consolidation: migrate 5 on-prem DBs to single Aurora cluster | Multiple DMS tasks, each migrating one DB — parallel migration, consolidated target |
| Development team needs a copy of prod data in dev (anonymized) | DMS with transformation rules — copy data and mask PII fields during migration |

**The SCT rule:** Homogeneous = same engine (MySQL → MySQL, Oracle → Oracle) — SCT NOT needed. Heterogeneous = different engines (Oracle → Aurora, SQL Server → PostgreSQL) — SCT REQUIRED.

---

## AWS Migration Services

### AWS Application Migration Service (MGN)
- Lift-and-shift (rehost) migration
- Replicate servers to AWS using **Replication Agent**
- Formerly CloudEndure Migration
- Supports: physical servers, VMware, Hyper-V, Azure, GCP

### AWS Server Migration Service (SMS)
- **Legacy** (use MGN instead)
- Replicate VMware/Hyper-V/Azure VMs to AMIs

### AWS Application Discovery Service
- Discover on-premises servers and applications
- Collect: server utilization, network dependencies, software inventory
- **Agentless Discovery:** VMware vCenter, no agent needed
- **Agent-based Discovery:** More detail, any OS
- Data fed into **AWS Migration Hub**

### AWS Migration Hub
- Central place to track all migration projects
- Integrates with MGN, DMS, Application Discovery Service

---

## Snow Family (Recap)

| Device | Storage | Use Case |
|--------|---------|---------|
| **Snowcone** | 8TB HDD/14TB SSD | Edge computing, small transfers |
| **Snowball Edge Storage** | 80TB | Large data migration |
| **Snowball Edge Compute** | 42TB + compute | Edge computing + data transfer |
| **Snowmobile** | 100PB | Exabyte migration (truck) |

### When to Use Snow vs Network Transfer

| Data Size | 1Gbps Connection | 10Gbps Connection | Recommendation |
|-----------|-----------------|------------------|----------------|
| 1TB | ~2.5 hours | ~15 min | Network |
| 10TB | ~25 hours | ~2.5 hours | Network or Snowcone |
| 100TB | ~10 days | ~1 day | Snowball |
| 1PB | ~3 months | ~12 days | Snowball (multiple) |
| 10PB+ | Years | Months | Snowmobile |

---

## CloudFormation

- **Infrastructure as Code (IaC)**
- Define resources in JSON or YAML templates
- **Stacks:** Group of AWS resources managed as a unit
- **Change Sets:** Preview changes before applying
- **Nested Stacks:** Reuse templates within templates
- **Stack Sets:** Deploy stacks across multiple accounts/regions
- **Drift Detection:** Detect manual changes to stack resources

### Key Concepts
- `Parameters`: Input values at stack creation
- `Mappings`: Static key-value mappings (region-specific config)
- `Conditions`: Create resources based on conditions
- `Outputs`: Export values for cross-stack references
- `DependsOn`: Explicit resource ordering
- `DeletionPolicy`: Retain, Delete, or Snapshot on stack deletion

### CloudFormation vs Terraform
- CloudFormation = AWS native, free
- Terraform = multi-cloud, OSS, more flexible

### Real-World Use Cases & When to Use CloudFormation

> **The problem it solves:** Define your entire AWS infrastructure as code — version controlled, repeatable, automated, and self-documenting. One template creates identical environments in minutes.

**Real-World Scenarios:**
| Business Problem | CloudFormation Solution |
|-----------------|------------------------|
| Team creates dev/staging/prod environments manually — configuration drift everywhere | CloudFormation Stack — one template, parameterized (env=dev/prod), identical every time |
| Enterprise needs to deploy the same 3-tier app in 10 AWS accounts | **Stack Sets** — deploy one stack to 10 accounts across regions with one command |
| Team accidentally deleted security group — need to track manual changes | **Drift Detection** — compares actual resource state vs template, flags changes |
| Company builds reusable VPC module for all teams | **Nested Stacks** — VPC template imported by all application stacks |
| New stack deployment could break production — need to preview changes | **Change Sets** — see what will be created/modified/deleted before executing |
| Database must survive stack deletion (production data protection) | `DeletionPolicy: Retain` on RDS resource — stack deletes but DB persists |

---

## AWS CDK — Cloud Development Kit

- Define infrastructure using programming languages (TypeScript, Python, Java, C#)
- Compiles to CloudFormation templates
- **Constructs:** Reusable components (L1=raw CFN, L2=opinionated, L3=patterns)

---

## Elastic Disaster Recovery (EDR)

- Replicate on-premises or cloud servers to AWS for DR
- Formerly CloudEndure Disaster Recovery
- RPO: seconds, RTO: minutes
- Continuous block-level replication

---

## AWS DataSync

- **Fast data transfer** between on-premises and AWS
- Supports: NFS, SMB, HDFS, S3, EFS, FSx
- **Agent-based** for on-premises sources
- Scheduled transfers, data integrity verification
- Up to **10x faster than open-source tools**

### DataSync vs Storage Gateway vs Snow

| Service | Use Case |
|---------|---------|
| **DataSync** | One-time or scheduled data migration to/from AWS |
| **Storage Gateway** | Ongoing hybrid access (on-prem apps access AWS storage) |
| **Snow Family** | Offline data transfer (too much data for network) |

### Real-World Use Cases & When to Use DataSync

> **The problem it solves:** Fast, automated data transfer between on-premises storage and AWS — up to 10x faster than open-source tools, with built-in integrity verification.

**Real-World Scenarios:**
| Business Problem | DataSync Solution |
|-----------------|-----------------|
| Data center shutting down — 50TB NFS share needs to move to EFS | DataSync with NFS agent on-premises — scheduled transfer, integrity check, 10x faster than rsync |
| Research institution transfers 100GB genomics datasets nightly to S3 | DataSync scheduled task — runs every night at 2 AM, only transfers changed files (incremental) |
| Migrate HDFS cluster data to S3 for a cloud data lake | DataSync HDFS connector — scan HDFS, transfer to S3, verify checksums |
| Company wants to migrate on-premises SMB file server to FSx for Windows | DataSync SMB agent → FSx for Windows — preserves NTFS metadata, ACLs |

**DataSync vs Storage Gateway for the exam:** DataSync = one-time or periodic migration/transfer. Storage Gateway = ongoing, permanent hybrid access where on-premises apps continue using AWS storage.

---

## AWS Transfer Family

- Managed file transfer using **SFTP, FTPS, FTP, AS2**
- Endpoint stores files in **S3 or EFS**
- Existing SFTP users/clients — no code changes needed
- Authentication: service-managed, AD, LDAP, custom Lambda

---

## AWS Outposts

- AWS infrastructure on-premises (rack or servers)
- Same AWS APIs, console, tools
- Local processing with low latency to on-premises systems
- Supports: EC2, EBS, S3, RDS, ECS, EKS, EMR

---

## AWS Local Zones

- Extension of AWS region to specific cities
- Single-digit millisecond latency to metro areas
- Subset of services available (EC2, EBS, ALB, RDS)
- Use for: media rendering, real-time gaming, financial trading

---

## AWS Wavelength

- AWS infrastructure inside 5G carrier networks
- Ultra-low latency for mobile/5G applications (milliseconds)
- Same AWS APIs
- Use for: autonomous vehicles, live streaming, AR/VR

---

## Migration Strategy — The 7 Rs

| Strategy | Description | Effort |
|---------|-------------|--------|
| **Retire** | Turn off (not needed) | None |
| **Retain** | Keep on-premises for now | None |
| **Rehost (Lift & Shift)** | Move to cloud without changes (MGN) | Low |
| **Replatform (Lift & Tinker)** | Minor optimizations (RDS instead of EC2 MySQL) | Medium |
| **Repurchase (Drop & Shop)** | Move to SaaS (Salesforce instead of CRM) | Medium |
| **Refactor/Re-architect** | Cloud-native redesign (microservices) | High |
| **Relocate** | VMware to VMware Cloud on AWS | Low-Medium |

### Real-World 7 Rs Examples

> **The problem it solves:** Choose the right migration strategy for each application — not every app needs to be refactored; some should just be moved, optimized slightly, or retired.

| Real-World Application | 7R Strategy | Rationale |
|-----------------------|------------|-----------|
| 100 old internal tools nobody uses | **Retire** | Discovery shows 30% of apps have zero traffic — decommission |
| Mainframe billing system — too risky to touch | **Retain** | Keep on-prem 2 more years while planning modernization |
| Legacy Windows app (no source code) | **Rehost** — AWS MGN | Lift entire VM to EC2, no changes needed |
| MySQL on EC2 (own server) | **Replatform** — RDS MySQL | Same MySQL, but now managed: automated backups, Multi-AZ, patching |
| On-premises CRM (custom-built) | **Repurchase** — Salesforce | Cheaper to buy SaaS than maintain custom CRM |
| Monolithic e-commerce app (scaling issues) | **Refactor** — microservices | Break into Lambda + DynamoDB + SQS for infinite scale |
| VMware on-premises vSphere cluster | **Relocate** — VMware Cloud on AWS | Same vSphere tools, move VMs to AWS without OS changes |

---

## Key Exam Traps

1. **RPO determines backup frequency** — 1-hour RPO means backup every hour
2. **Pilot Light ≠ Warm Standby** — Pilot Light: DB running, EC2 stopped. Warm: both running at min
3. **DMS requires Replication Instance** — it's not serverless; you provision an EC2-like instance
4. **SCT needed for heterogeneous** DB migrations only
5. **DataSync = data migration tool**, NOT a continuous sync (Storage Gateway is for ongoing hybrid)
6. **CloudFormation DeletionPolicy=Retain** — resource persists even if stack is deleted
7. **StackSets require trusted access** with AWS Organizations or self-managed permissions
8. **Snow devices are shipped physically** — plan for transit time (1-2 weeks) in your RPO calculations
