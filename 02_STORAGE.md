# Storage Services — SAA-C03 Exam Guide

---

## S3 — Simple Storage Service (HIGHEST EXAM WEIGHT)

### Core Concepts
- Object storage, max object size: **5TB** (multipart upload required > 5GB)
- Bucket names globally unique, region-specific
- Default: **private**, all public access blocked
- Unlimited storage capacity
- Strong read-after-write consistency (since Dec 2020)

### S3 Storage Classes (MEMORIZE ALL)
| Storage Class | Availability | Retrieval | Use Case |
|--------------|-------------|-----------|---------|
| **S3 Standard** | 99.99% | Immediate | Active, frequently accessed |
| **S3 Standard-IA** | 99.9% | Immediate | Infrequently accessed, min 30 days |
| **S3 One Zone-IA** | 99.5% | Immediate | Non-critical, one AZ, min 30 days |
| **S3 Intelligent-Tiering** | 99.9% | Immediate | Unknown/changing access patterns |
| **S3 Glacier Instant** | 99.9% | Milliseconds | Archive, quarterly access, min 90 days |
| **S3 Glacier Flexible** | 99.99% | Min/Hours | Archive, min 90 days |
| **S3 Glacier Deep Archive** | 99.99% | 12-48 hours | Long-term, rarely accessed, min 180 days |

> **Exam Tip:** Minimum storage duration charges: Standard-IA=30d, Glacier Instant=90d, Glacier Flexible=90d, Glacier Deep Archive=180d

### S3 Versioning
- Once enabled, cannot be disabled (only suspended)
- Delete marker for versioned delete
- MFA Delete — requires MFA to delete versions or disable versioning
- Previous versions remain and incur storage costs

### S3 Lifecycle Policies
- Transition between storage classes based on age
- Expire objects (delete after N days)
- Delete incomplete multipart uploads
- Can apply to current/previous versions separately

### S3 Replication
| | Cross-Region (CRR) | Same-Region (SRR) |
|-|-------------------|-------------------|
| Use Case | Compliance, disaster recovery, latency | Log aggregation, test with prod data |
| Requirement | Versioning must be enabled on source AND destination |
| Replication | New objects only (not existing) |
| Delete markers | Optional replication |

### S3 Security
- **Bucket Policy** — resource-based, JSON, cross-account access
- **ACL** — legacy, object/bucket level (avoid for new setups)
- **Block Public Access** — account-level or bucket-level setting
- **Presigned URLs** — temporary access (up to 7 days via SDK, 12hr via console)
- **S3 Access Points** — simplified access management for shared datasets
- **S3 Object Lock:** WORM (Write Once Read Many)
  - Compliance mode: cannot be deleted even by root
  - Governance mode: can be overridden with special permissions
  - Retention period or Legal Hold

### S3 Encryption
| Type | Description |
|------|-------------|
| **SSE-S3** | AWS-managed keys (AES-256), default |
| **SSE-KMS** | KMS keys, audit trail, more control |
| **SSE-C** | Customer-provided keys, client manages |
| **Client-side** | Encrypt before upload |

> **Exam Tip:** Enforce encryption via bucket policy denying `s3:PutObject` without encryption header

### S3 Performance
- **Multipart Upload:** Recommended > 100MB, required > 5GB
- **S3 Transfer Acceleration:** CloudFront edge network for uploads
- **Byte-Range Fetches:** Parallel downloads for large files
- 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD per second per prefix
- Use random prefixes to maximize throughput (no longer needed — AWS resolved)

### S3 Event Notifications
- Triggers: ObjectCreated, ObjectRemoved, ObjectRestore, Replication events
- Destinations: Lambda, SNS, SQS, EventBridge
- EventBridge offers most filtering options

### S3 Website Hosting
- Static website hosting: bucket must be public
- Custom domain via Route 53 (bucket name must match domain)
- CORS configuration for cross-origin requests

### Advanced S3 Features
- **S3 Select:** SQL queries on S3 objects (CSV, JSON, Parquet)
- **Glacier Select:** SQL queries on Glacier archives
- **S3 Inventory:** List objects and metadata (CSV, ORC, Parquet)
- **S3 Analytics:** Analyze access patterns for lifecycle decisions
- **Requester Pays:** Requester pays data transfer costs (not bucket owner)

---

## EBS — Elastic Block Store

### Volume Types (MEMORIZE)
| Type | Category | Max IOPS | Max Throughput | Use Case |
|------|---------|---------|---------------|---------|
| **gp3** | SSD General | 16,000 | 1,000 MB/s | Boot, general workloads (DEFAULT) |
| **gp2** | SSD General | 16,000 | 250 MB/s | Legacy general purpose |
| **io1** | SSD Provisioned IOPS | 64,000 | 1,000 MB/s | Critical DBs, > 16K IOPS |
| **io2 Block Express** | SSD Provisioned IOPS | 256,000 | 4,000 MB/s | SAP HANA, largest DBs |
| **st1** | HDD Throughput | 500 IOPS | 500 MB/s | Big data, log processing |
| **sc1** | HDD Cold | 250 IOPS | 250 MB/s | Archival, infrequent access |

> **Exam Tip:** Only SSD (gp2, gp3, io1, io2) can be boot volumes. HDD cannot be boot volumes.

### EBS Key Facts
- **Single AZ only** — cannot attach to EC2 in different AZ
- **Multi-Attach (io1/io2):** Attach to up to 16 Nitro EC2s in same AZ (cluster apps)
- Can take snapshots → stored in S3 (managed), incremental
- Snapshots can be copied across regions
- Encryption: encrypted volumes create encrypted snapshots (and vice versa)
- Cannot decrease EBS size (can only increase)
- gp3: IOPS and throughput independently configurable

### EBS Snapshots
- Incremental (only changed blocks)
- Snapshot while running (slightly inconsistent), better to stop
- **Fast Snapshot Restore (FSR):** Eliminates initialization latency (costly)
- **Data Lifecycle Manager (DLM):** Automate snapshot policies
- **Recycle Bin:** Recover accidentally deleted snapshots

---

## EFS — Elastic File System

- Managed NFS (Network File System) — **Linux only**
- Multi-AZ, multi-instance shared access (thousands of concurrent)
- Automatically scales, no capacity planning needed
- Pay for what you use
- **3x more expensive than EBS gp2**

### EFS Performance Modes
| Mode | Use Case |
|------|---------|
| **General Purpose** (default) | Low latency, web, CMS |
| **Max I/O** | High parallelism, big data, media processing |

### EFS Throughput Modes
| Mode | Description |
|------|-------------|
| **Bursting** | Scales with storage size |
| **Provisioned** | Set throughput independent of size |
| **Elastic** | Auto-scales up/down (recommended) |

### EFS Storage Classes
- **Standard:** Active files
- **Standard-IA (Infrequent Access):** Cost savings for infrequently accessed
- **One Zone / One Zone-IA:** Single AZ, cheaper, lower durability

### EFS Security
- Mount targets per AZ
- Security groups control access
- IAM + EFS access points for per-directory permissions
- Encryption in transit (TLS) and at rest (KMS)

---

## FSx Family

### FSx for Windows File Server
- SMB protocol, NTFS
- Windows ACLs, DFS namespaces
- Active Directory integration
- Multi-AZ for HA
- Use for: Windows workloads, SharePoint, SQL Server, home directories

### FSx for Lustre
- High-performance parallel file system
- Sub-millisecond latency, hundreds of GB/s throughput
- Integration with S3 (read from / write to S3)
- Use for: HPC, ML, financial modeling, video rendering
- Deployment: Scratch (no replication, short-term) or Persistent (replicated)

### FSx for NetApp ONTAP
- NFS, SMB, iSCSI protocol support
- Multi-protocol, snapshots, replication, deduplication
- Use for: Lift-and-shift NetApp workloads

### FSx for OpenZFS
- NFS protocol, Linux/macOS/Windows
- Snapshots, data cloning
- Use for: Migrate ZFS workloads to AWS

---

## Storage Gateway

Hybrid cloud storage — connect on-premises to AWS storage.

| Gateway Type | Protocol | Backend | Use Case |
|-------------|---------|---------|---------|
| **S3 File Gateway** | NFS/SMB | S3 (any storage class) | File shares → S3, replace NAS |
| **FSx File Gateway** | SMB | FSx for Windows | Windows file shares, low-latency cache |
| **Volume Gateway (Cached)** | iSCSI | S3 + local cache | Primary data in S3, cache locally |
| **Volume Gateway (Stored)** | iSCSI | Local + async S3 backup | Primary local, backup to S3 |
| **Tape Gateway** | iSCSI VTL | S3 + Glacier | Replace physical tape library |

> **Exam Tip:** Storage Gateway = hybrid (on-prem to cloud). File Gateway = NFS/SMB to S3. Tape Gateway = virtual tape library.

---

## Snow Family

| Device | Storage | Use Case |
|--------|---------|---------|
| **Snowcone** | 8TB HDD / 14TB SSD | Small, portable, edge computing |
| **Snowball Edge Storage** | 80TB | Large data migration, no compute |
| **Snowball Edge Compute** | 42TB | Edge computing + storage |
| **Snowmobile** | 100PB | Exabyte-scale migration (truck) |

- Use when: >1 week to transfer over network, or bandwidth limited
- **OpsHub:** GUI to manage Snow devices
- Snow devices support EC2 compute (Lambda on Snowcone)

> **Exam Tip:** 10Gbps connection, 100TB data → ~11 days → use Snowball. 1Gbps, 100TB → ~111 days → definitely Snowball.

---

## Storage Comparison (Exam Cheat Sheet)

| Storage | Type | Use Case | Multi-instance |
|---------|------|---------|---------------|
| **S3** | Object | Media, backups, static websites | Yes |
| **EBS** | Block | EC2 boot, databases | No (except Multi-Attach io1/io2) |
| **EFS** | File (NFS) | Shared Linux file system | Yes |
| **FSx Windows** | File (SMB) | Shared Windows file system | Yes |
| **FSx Lustre** | File (HPC) | High-performance computing | Yes |
| **Instance Store** | Block (ephemeral) | Temp data, cache, buffers | No |

---

## Key Exam Traps

1. **S3 is not a file system** — you can't mount it (use Storage Gateway for file access)
2. **EBS only single AZ** — can't span AZs; use EFS for multi-AZ shared storage
3. **EFS = Linux only** — for Windows shared storage, use FSx for Windows
4. **S3 Standard-IA minimum charge** = 30 days storage + retrieval fee
5. **SSE-KMS adds API call costs** — high-volume applications consider SSE-S3
6. **Glacier ≠ immediate access** — use Glacier Instant Retrieval if you need millisecond access
7. **Presigned URLs** — anyone with the URL can access the object until expiry
