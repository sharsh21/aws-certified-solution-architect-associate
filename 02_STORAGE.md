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

### Real-World Use Cases & When to Choose S3

> **The problem it solves:** Infinitely scalable, durable object storage for any type of file — the backbone of most AWS architectures for backups, data lakes, static websites, and file distribution.

**Choose S3 when:** You need to store files/objects of any size, share them across services, or build a cost-tiered archive strategy.

**Real-World Scenarios:**
| Business Problem | S3 Solution |
|-----------------|------------|
| Netflix-scale video platform needs to store petabytes of content | S3 Standard for hot content → Lifecycle to S3 Standard-IA after 30 days → Glacier after 1 year |
| Healthcare company must retain patient records for 10 years, rarely accessed | S3 Glacier Deep Archive — $0.00099/GB/month, compliant, durable |
| SaaS company serves static React app globally | S3 static website hosting + CloudFront CDN — fast, cheap, no servers |
| Compliance team needs audit logs that can NEVER be deleted | S3 Object Lock (Compliance mode) + MFA Delete — even root can't delete |
| Data science team queries 5TB of CSV logs daily | S3 + Athena — query in-place with SQL, pay per TB scanned (no ETL) |
| Multi-region DR: primary region S3 data must be in EU backup | Cross-Region Replication (CRR) with versioning enabled |
| Mobile app where each user accesses only their own files | S3 with IAM Identity Center or Cognito Identity Pools + bucket policies scoped per user |

**Storage class decision rule for the exam:**
- Accessed daily → Standard
- Accessed monthly, cost-sensitive → Standard-IA
- Archive, millisecond access needed → Glacier Instant Retrieval
- Archive, 5-12 hour retrieval OK → Glacier Flexible
- Compliance archive, 7+ years → Glacier Deep Archive

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

### Real-World Use Cases & When to Choose EBS

> **The problem it solves:** Persistent block storage attached to EC2 — like a hard drive in the cloud. Required for databases, boot volumes, and any workload needing low-latency block I/O.

**Choose EBS when:** EC2 instance needs a persistent disk (database, application files, boot volume). Only one instance needs the volume at a time (exception: Multi-Attach io1/io2).

**Real-World Scenarios:**
| Business Problem | EBS Solution |
|-----------------|-------------|
| MySQL database on EC2 needs fast, reliable storage | **gp3** — 16,000 IOPS, 1,000 MB/s, cost-effective SSD for transactional DBs |
| Oracle DB needs guaranteed 50,000 IOPS (SLA requirement) | **io1/io2** — provisioned IOPS, guaranteed performance regardless of load |
| Hadoop cluster on EC2 processes 500TB of log data | **st1 (HDD throughput)** — sequential reads, cheap per GB, great for MapReduce |
| Infrequent archive files on EC2 (accessed once a month) | **sc1 (Cold HDD)** — lowest cost EBS, fine for rarely accessed data |
| EC2 crash leaves database in unknown state — need point-in-time recovery | EBS Snapshots (incremental, stored in S3) + DLM to automate daily snapshots |
| Oracle RAC cluster needs shared block storage across 2 nodes in same AZ | **io2 Multi-Attach** — up to 16 Nitro EC2 instances share one volume |

**Volume type decision rule:** Boot/general use → gp3. High IOPS DB (> 16K IOPS) → io2. Sequential big data → st1. Cold archive on EC2 → sc1.

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

### Real-World Use Cases & When to Choose EFS

> **The problem it solves:** Shared file storage accessible by thousands of Linux EC2 instances simultaneously — like an NFS server in the cloud that auto-scales and requires zero capacity planning.

**Choose EFS when:** Multiple EC2 instances (or containers) need to read/write the same files simultaneously. Linux workloads only.

**Real-World Scenarios:**
| Business Problem | EFS Solution |
|-----------------|-------------|
| WordPress farm: 20 EC2 web servers need to share `/wp-content/uploads` | EFS mounted on all web servers — one shared media directory, no sync issues |
| CI/CD system: 50 Jenkins build agents need a shared workspace | EFS in Max I/O mode — high parallelism, all agents read/write build artifacts |
| Containerized app on ECS needs persistent shared storage across tasks | EFS volume mount in ECS task definition — Fargate tasks share the same files |
| Application reads config files that must be identical across all instances | EFS — single source of truth, all instances see the same filesystem |
| ML training: multiple nodes need access to the same training dataset | EFS in Max I/O — all training nodes read dataset simultaneously |

**EFS vs EBS for the exam:** EFS = shared, multi-AZ, multi-instance, Linux, expensive. EBS = single instance, one AZ, any OS, cheaper.

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

### Real-World Use Cases & When to Choose FSx

> **The problem it solves:** Managed, high-performance file systems for specialized workloads — Windows shares, HPC, NetApp migrations — where EFS (NFS only) doesn't fit.

**FSx decision table for the exam:**
| Your Situation | Choose |
|---------------|-------|
| Windows users need shared drives (SMB, NTFS, Active Directory) | **FSx for Windows File Server** |
| HPC/ML workloads need sub-millisecond, hundreds of GB/s throughput | **FSx for Lustre** |
| Migrating existing NetApp ONTAP storage to AWS (NFS + SMB + iSCSI) | **FSx for NetApp ONTAP** |
| Migrating ZFS-based workloads | **FSx for OpenZFS** |

**Real-World Scenarios:**
| Business Problem | FSx Solution |
|-----------------|-------------|
| Law firm migrates 50TB Windows file shares to AWS (DFS namespaces, AD auth) | FSx for Windows File Server — native SMB, NTFS ACLs, AD join |
| Rendering studio needs 500 GB/s throughput for 3D animation pipeline | FSx for Lustre — feeds petabytes to GPU render farm at wire speed |
| Financial firm lifts-and-shifts NetApp SAN to AWS (runs NFS + iSCSI) | FSx for NetApp ONTAP — same protocols, same tools, dedup/compression |
| ML training: 100 GPU nodes need fast access to 10TB dataset in S3 | FSx for Lustre — links to S3 directly, auto-imports data, sub-ms reads |

**Key exam trap:** FSx for Windows = SMB (not NFS). EFS = NFS (not SMB). If the question says "Windows" or "Active Directory" → FSx for Windows, not EFS.

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

### Real-World Use Cases & When to Choose Storage Gateway

> **The problem it solves:** Seamlessly extend on-premises storage to AWS cloud — your on-premises apps use standard protocols (NFS, SMB, iSCSI) while data is stored in AWS behind the scenes.

**Choose Storage Gateway when:** On-premises apps can't be refactored but you want to back up or extend to AWS. Hybrid is permanent, not just a migration.

**Real-World Scenarios:**
| Business Problem | Storage Gateway Solution |
|-----------------|------------------------|
| Branch offices have NAS devices — IT wants to back them all up to S3 centrally | **S3 File Gateway** — each branch mounts NFS share, files land in S3 automatically |
| Existing Windows file servers use DFS — company wants S3 as backend | **FSx File Gateway** — SMB protocol, local cache for low-latency, syncs to FSx for Windows |
| SAP on EC2 uses iSCSI block storage that must also be backed up to S3 | **Volume Gateway (Stored)** — primary storage stays on-premises, S3 is async backup |
| Hospital has 5TB of active patient images on iSCSI SAN — overflow to cloud | **Volume Gateway (Cached)** — primary data in S3, hot files cached locally (< 20% of data) |
| Broadcast company has 20 physical tape libraries — very expensive to maintain | **Tape Gateway** — replaces physical tapes with virtual tapes in S3 + Glacier |

**The exam pattern:** On-premises NFS/SMB → S3 behind the scenes = **File Gateway**. Physical tape replacement = **Tape Gateway**. On-premises iSCSI block storage extension = **Volume Gateway**.

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

### Real-World Use Cases & When to Choose Snow Family

> **The problem it solves:** Physically transport massive amounts of data to/from AWS when network bandwidth makes it impractical — or process data at the edge where there is no internet connection.

**The rule of thumb:** If transferring over the network would take > 1 week, use Snow. If > 1 PB, use Snowmobile.

**Real-World Scenarios:**
| Business Problem | Snow Solution |
|-----------------|--------------|
| Oil rig collects 5TB of seismic data daily — no reliable internet | **Snowcone** — ruggedized, tiny (2.1 kg), upload data when ship returns to port |
| Data center migration: 500TB of legacy data needs to move to AWS | **Snowball Edge Storage** — ship 7 devices, load data, ship back — done in weeks |
| Disaster response team needs AWS compute in a remote area with no internet | **Snowball Edge Compute** — runs EC2 + Lambda at the edge, stores data locally |
| Media company needs to move 40PB archive from decommissioned data center | **Snowmobile** — AWS drives a truck to your data center, semi-trailer holds 100PB |
| Military needs secure AWS compute in classified facility (no cloud access) | **Snowball Edge Compute** — air-gapped, tamper-resistant, ITAR-compliant |

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
