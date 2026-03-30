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
