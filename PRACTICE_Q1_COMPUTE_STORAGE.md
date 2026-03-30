# Practice Questions — Compute & Storage (Q1–Q28)

> Format: Question → Options → Answer → Explanation
> Difficulty: Mix of Easy / Medium / Hard (marked as E/M/H)

---

### Q1 (M) — EC2 Purchasing
A company runs a batch processing workload every weekend for 6 hours. The job is fault-tolerant and can be restarted if interrupted. Which EC2 purchasing option is MOST cost-effective?

- A) On-Demand Instances
- B) Reserved Instances (1-year, No Upfront)
- C) Spot Instances
- D) Dedicated Hosts

**Answer: C**
> Spot Instances offer up to 90% discount and are ideal for fault-tolerant, flexible workloads that can handle interruptions. Batch jobs that can restart are the classic Spot use case.

---

### Q2 (M) — EC2 Storage
A developer needs maximum I/O performance for a temporary database cache on an EC2 instance. The data does NOT need to persist after the instance stops. What is the best storage option?

- A) EBS gp3 volume
- B) EBS io2 volume
- C) EC2 Instance Store
- D) EFS with Max I/O mode

**Answer: C**
> Instance Store provides the highest IOPS (hardware-level NVMe) and lowest latency because it's physically attached to the host. Since persistence is not required, it's the optimal choice. EBS and EFS add network overhead.

---

### Q3 (H) — Auto Scaling
An application experiences unpredictable traffic spikes. The Auto Scaling Group currently uses a Target Tracking policy set to 60% CPU. During a sudden spike, new instances take 8 minutes to become healthy. Users experience degraded performance during this period. What should the architect recommend?

- A) Switch to Step Scaling with a lower CPU threshold
- B) Enable Predictive Scaling in addition to Target Tracking
- C) Increase the desired capacity permanently
- D) Use Scheduled Scaling with a fixed schedule

**Answer: B**
> Predictive Scaling uses ML to forecast traffic and pre-launches instances BEFORE the spike occurs, eliminating the warm-up lag. Target Tracking reacts after the spike — Predictive acts proactively. Increasing desired capacity permanently wastes money.

---

### Q4 (M) — Load Balancer Selection
A gaming company needs a load balancer that supports TCP/UDP traffic, provides a static IP address per Availability Zone, and handles millions of requests per second with ultra-low latency. Which load balancer should be used?

- A) Application Load Balancer
- B) Classic Load Balancer
- C) Network Load Balancer
- D) Gateway Load Balancer

**Answer: C**
> NLB operates at Layer 4 (TCP/UDP), supports static/Elastic IPs per AZ, and handles millions of RPS with sub-millisecond latency. ALB is Layer 7 (HTTP) and doesn't support static IPs or UDP.

---

### Q5 (M) — Lambda Limits
A Lambda function processes video files uploaded to S3. Some videos are up to 3GB. The function downloads the file, processes it, and uploads the result. The function is timing out at 15 minutes. What should the architect do?

- A) Increase the Lambda timeout beyond 15 minutes
- B) Migrate the function to ECS/Fargate with a longer-running container task
- C) Use Lambda Provisioned Concurrency to speed up execution
- D) Increase Lambda memory to maximum (10,240 MB)

**Answer: B**
> Lambda has a hard maximum timeout of 15 minutes — it cannot be increased. For long-running processes, migrate to ECS/Fargate (no time limit) or EC2. Provisioned Concurrency reduces cold starts, not execution time.

---

### Q6 (E) — Lambda Cold Start
A company uses Java-based Lambda functions behind API Gateway. Users occasionally report high latency on the first request. What is the MOST effective solution?

- A) Increase the Lambda function memory allocation
- B) Enable Lambda Provisioned Concurrency
- C) Enable Lambda Reserved Concurrency
- D) Use a Kinesis trigger instead of API Gateway

**Answer: B**
> Provisioned Concurrency pre-initializes Lambda execution environments, eliminating cold starts. Reserved Concurrency sets a maximum limit but doesn't prevent cold starts. Memory increase speeds execution but doesn't prevent initialization latency.

---

### Q7 (M) — ECS vs EKS
A startup wants to containerize their application with minimal operational overhead. Their team has no Kubernetes experience. Which compute option is MOST appropriate?

- A) EKS with managed node groups
- B) ECS with Fargate
- C) EKS with Fargate
- D) EC2 with Docker Compose

**Answer: B**
> ECS with Fargate is the simplest container option — no Kubernetes knowledge needed, no EC2 infrastructure to manage. EKS requires Kubernetes expertise. Fargate removes the need to manage underlying EC2 instances.

---

### Q8 (H) — Placement Groups
A financial services company runs a high-performance computing (HPC) cluster that requires extremely low network latency between nodes and high throughput. The cluster has 20 instances. Which placement group type should be used?

- A) Spread placement group
- B) Partition placement group
- C) Cluster placement group
- D) No placement group needed

**Answer: C**
> Cluster placement groups place instances close together in a single AZ on the same physical hardware, providing the lowest network latency and highest throughput. Spread has max 7 instances per AZ. Partition is for distributed systems (Kafka, Cassandra), not HPC.

---

### Q9 (M) — S3 Storage Classes
A company stores compliance documents that must be retained for 7 years. The documents are accessed frequently in the first 30 days, rarely accessed for the next 6 months, then almost never accessed. Which lifecycle policy is MOST cost-effective?

- A) Store in S3 Standard for 7 years
- B) S3 Standard (0-30 days) → S3 Standard-IA (30-180 days) → S3 Glacier Deep Archive (180+ days)
- C) S3 Standard (0-30 days) → S3 Glacier Flexible (30+ days)
- D) Use S3 Intelligent-Tiering for all 7 years

**Answer: B**
> This lifecycle matches access patterns perfectly: Standard for active access, Standard-IA for infrequent access (cheaper storage + retrieval fee), Glacier Deep Archive for long-term near-zero access at the lowest storage cost. Intelligent-Tiering adds monthly monitoring fees per object.

---

### Q10 (H) — S3 Security
A company wants to allow only encrypted uploads to an S3 bucket. Uploads without encryption should be rejected. How should this be implemented?

- A) Enable default encryption on the bucket
- B) Add a bucket policy with a Deny condition for requests without `x-amz-server-side-encryption` header
- C) Enable S3 Block Public Access
- D) Use an S3 ACL to restrict unencrypted uploads

**Answer: B**
> Default encryption encrypts objects even if uploaded without the header (it doesn't reject them). To actively REJECT unencrypted uploads, you need a bucket policy with `Deny` on `s3:PutObject` when the encryption header is absent. ACLs cannot enforce encryption.

---

### Q11 (M) — S3 Replication
A company replicates S3 objects from us-east-1 to eu-west-1 for disaster recovery. After enabling Cross-Region Replication (CRR), they notice existing objects were not replicated. What should they do?

- A) Disable and re-enable CRR
- B) Use S3 Batch Operations to replicate existing objects
- C) Enable versioning on the destination bucket
- D) Use AWS DataSync to copy existing objects

**Answer: B**
> CRR only replicates NEW objects added after replication is configured. Existing objects must be replicated using **S3 Batch Operations** with a replication job. DataSync works but is an extra cost; S3 Batch Operations is the AWS-native solution.

---

### Q12 (E) — EBS Volume Types
A database workload requires consistent 50,000 IOPS with sub-millisecond latency. Which EBS volume type should be used?

- A) gp3
- B) gp2
- C) io2
- D) st1

**Answer: C**
> io2 (Provisioned IOPS SSD) supports up to 64,000 IOPS (io2 Block Express: 256,000). gp3 maxes at 16,000 IOPS. st1 is HDD — not suitable for database IOPS. io2 is designed for I/O-intensive databases (Oracle, SQL Server, SAP).

---

### Q13 (M) — EFS vs EBS
A company has 10 EC2 instances running a Linux web application that need to share a common file system for user-uploaded content. The solution must scale automatically and be highly available across multiple AZs. What should be used?

- A) EBS gp3 volume with Multi-Attach enabled
- B) Amazon EFS
- C) S3 mounted with s3fs
- D) EBS snapshot shared across instances

**Answer: B**
> EFS is a managed NFS that natively supports multiple EC2 instances across multiple AZs simultaneously. It auto-scales. EBS Multi-Attach is limited to 16 Nitro instances in the SAME AZ only. s3fs works but is not a native file system and has performance limitations.

---

### Q14 (H) — Storage Gateway
A company has an on-premises backup application that uses tape drives. They want to eliminate physical tapes and store backups in AWS Glacier instead. The backup application must not require modification. What should be used?

- A) S3 File Gateway
- B) Volume Gateway (Stored mode)
- C) Tape Gateway
- D) AWS DataSync

**Answer: C**
> Tape Gateway presents a virtual tape library (VTL) using iSCSI — existing backup applications see it as physical tape hardware without any modification. Data is stored in S3 and archived to Glacier automatically. S3 File Gateway is NFS/SMB, not tape protocol.

---

### Q15 (M) — Snow Family
A company needs to migrate 80TB of data from an on-premises data center to S3. Their internet connection is 100 Mbps and is 60% utilized by business traffic. How long would a network transfer take, and what should they use instead?

- A) ~18 days via internet; use Snowball Edge Storage Optimized
- B) ~2 days via internet; use DataSync
- C) ~5 days via internet; use Direct Connect
- D) ~1 day via internet; use S3 Transfer Acceleration

**Answer: A**
> Available bandwidth: 100Mbps × 40% = 40Mbps = 5MB/s. 80TB / 5MB/s ≈ 185 days. That's far too long — use Snowball. Even at full 100Mbps: 80TB / 12.5MB/s ≈ 74 days. Snowball Edge Storage Optimized holds 80TB and is shipped physically, typically completing in 1-2 weeks total.

---

### Q16 (M) — S3 Presigned URLs
A company wants to allow customers to upload large video files directly to S3 without routing through their application server (to reduce server load). The uploads should be temporary and expire after 1 hour. What is the recommended approach?

- A) Make the S3 bucket public and provide the bucket URL
- B) Generate a presigned URL with a 1-hour expiry from the backend and return it to the client
- C) Create an IAM user for each customer and provide temporary credentials
- D) Use S3 Transfer Acceleration with a public endpoint

**Answer: B**
> Presigned URLs grant temporary, time-limited access to a specific S3 operation (PUT for uploads). The backend generates the URL using its IAM credentials. The client uploads directly to S3, reducing server load. No public access needed; the URL expires after 1 hour.

---

### Q17 (H) — S3 Object Lock
A financial institution must store transaction logs in S3 such that even the root user cannot delete or modify them for 5 years to meet regulatory compliance. How should this be configured?

- A) Enable S3 Versioning and MFA Delete
- B) Enable S3 Object Lock in Governance mode with 5-year retention
- C) Enable S3 Object Lock in Compliance mode with 5-year retention
- D) Apply a bucket policy denying `s3:DeleteObject` to all principals

**Answer: C**
> Object Lock in **Compliance mode** prevents deletion or modification by ANY user, including root, for the retention period. **Governance mode** can be overridden by users with special permissions. Bucket policies can be modified by root. MFA Delete can be bypassed by root.

---

### Q18 (M) — Elastic Beanstalk Deployment
A company uses Elastic Beanstalk for their web application. They need to deploy a new version with zero downtime and the ability to instantly roll back if issues occur. Cost is not a primary concern. Which deployment policy should be used?

- A) All at once
- B) Rolling
- C) Rolling with additional batch
- D) Immutable

**Answer: D**
> Immutable deployment creates entirely new instances with the new version. If the health check fails, the old instances are still running — instant rollback by just terminating new instances. Zero downtime guaranteed. It's the most expensive (double instances temporarily) but safest.

---

### Q19 (E) — Lambda Triggers
A company uploads images to S3. Each upload should trigger an image processing Lambda function. Which configuration achieves this?

- A) Configure S3 Event Notifications to trigger Lambda on `s3:ObjectCreated:*`
- B) Use CloudWatch Events to poll S3 every minute
- C) Configure SQS to receive S3 events and trigger Lambda
- D) Set up an API Gateway to invoke Lambda when S3 receives uploads

**Answer: A**
> S3 Event Notifications natively trigger Lambda on object creation events. This is direct, serverless, and event-driven. Option C (SQS → Lambda) would also work and adds reliability/retry, but direct S3 → Lambda is simpler and more direct for this use case.

---

### Q20 (M) — EC2 Instance Metadata
An application running on EC2 needs to determine which Availability Zone it is currently running in at runtime. How should the application retrieve this information?

- A) Store the AZ in an environment variable during deployment
- B) Call the EC2 Instance Metadata Service at `http://169.254.169.254/latest/meta-data/placement/availability-zone`
- C) Query the CloudWatch API
- D) Use the AWS CLI `describe-instances` command within the code

**Answer: B**
> The EC2 Instance Metadata Service (IMDS) is available at `169.254.169.254` from within the instance. It provides AZ, region, instance ID, IAM credentials, and more — at no cost, with no API calls or permissions required.

---

### Q21 (H) — ASG Lifecycle Hooks
A company uses Auto Scaling. When instances are being terminated, they need to flush cache data to S3 before the instance is fully terminated. How can this be achieved?

- A) Use a CloudWatch alarm to detect termination and trigger Lambda
- B) Use an ASG Lifecycle Hook on `EC2_INSTANCE_TERMINATING` state to delay termination and run a script
- C) Enable EC2 termination protection
- D) Use a scheduled Lambda to detect and flush instances every 5 minutes

**Answer: B**
> Lifecycle Hooks pause the instance in a wait state (`Terminating:Wait`) for up to 2 hours, allowing time to run custom actions (flush cache, drain connections, backup logs) before the instance is actually terminated. The hook completes when the action calls `complete-lifecycle-action`.

---

### Q22 (M) — EBS Encryption
A company creates an unencrypted EBS volume. They later realize it needs to be encrypted for compliance. What is the correct way to encrypt the existing unencrypted volume?

- A) Enable encryption on the existing volume via the console
- B) Create an unencrypted snapshot → copy the snapshot with encryption enabled → create a new encrypted volume from the encrypted snapshot
- C) Attach the volume to a new encrypted EC2 instance
- D) Use AWS KMS to encrypt the volume in-place

**Answer: B**
> You cannot encrypt an existing EBS volume in-place. The correct process is: (1) Create snapshot of unencrypted volume, (2) Copy snapshot with encryption enabled (specify KMS key), (3) Create new encrypted volume from the encrypted snapshot, (4) Replace old volume. There is no in-place encryption.

---

### Q23 (E) — S3 Transfer Acceleration
A global media company has users in Asia, Europe, and the US uploading large video files to an S3 bucket in us-east-1. Upload speeds are slow for international users. What should be used?

- A) Enable Cross-Region Replication
- B) Enable S3 Transfer Acceleration
- C) Use CloudFront with S3 as origin
- D) Create S3 buckets in each region

**Answer: B**
> S3 Transfer Acceleration uses CloudFront's edge network to accelerate uploads. Data is routed from the nearest edge location over AWS's optimized backbone network to S3. This significantly improves upload speeds for geographically distant users.

---

### Q24 (H) — EC2 Dedicated Hosts vs Dedicated Instances
A company uses Oracle Database software with a per-socket license. They must control which physical server their EC2 instance runs on for licensing compliance. What should they use?

- A) Dedicated Instances
- B) Dedicated Hosts
- C) Reserved Instances
- D) Cluster Placement Group

**Answer: B**
> Dedicated Hosts give you visibility and control over the physical server, including socket count and core count — required for BYOL (Bring Your Own License) per-socket/per-core licensing (Oracle, Windows Server, SQL Server). Dedicated Instances are isolated on dedicated hardware but you don't control the physical server placement.

---

### Q25 (M) — FSx Selection
A company needs a high-performance shared file system for Linux-based HPC workloads. The system must deliver sub-millisecond latency and hundreds of GB/s throughput. Which storage service should be used?

- A) Amazon EFS with Max I/O performance mode
- B) Amazon FSx for Lustre
- C) Amazon FSx for Windows File Server
- D) EBS io2 with Multi-Attach

**Answer: B**
> FSx for Lustre is purpose-built for HPC, ML, and high-throughput workloads. It delivers sub-millisecond latency and hundreds of GB/s throughput with millions of IOPS. EFS Max I/O is for high concurrency but higher latency. FSx for Windows is SMB-based, not for Linux HPC.

---

### Q26 (E) — Lambda + RDS
A Lambda function connects to an Amazon RDS MySQL database. Under high traffic, the database runs out of connections and throws "too many connections" errors. What is the recommended solution?

- A) Increase the RDS instance size
- B) Use Amazon RDS Proxy
- C) Increase Lambda reserved concurrency
- D) Switch to DynamoDB

**Answer: B**
> Lambda functions create a new database connection per invocation, and under high concurrency, this exhausts the database connection pool. RDS Proxy pools and reuses database connections, dramatically reducing the number of actual connections to RDS. This is the primary use case for RDS Proxy with Lambda.

---

### Q27 (M) — S3 Multipart Upload
An application uploads 10GB files to S3. Occasionally, uploads fail partway through and must restart from the beginning, causing long delays. What should be implemented?

- A) Enable S3 Transfer Acceleration
- B) Use the S3 Multipart Upload API and implement resume logic
- C) Compress files before uploading
- D) Use an S3 Batch Operations job

**Answer: B**
> Multipart Upload splits large files into parts that are uploaded independently and in parallel. If a part fails, only that part needs to be retried — not the entire file. It also enables higher throughput through parallel uploads. Required for objects > 5GB; recommended for > 100MB.

---

### Q28 (H) — EC2 Right-Sizing
A company wants to identify EC2 instances that are underutilized (oversized) across their AWS account to reduce costs. Which AWS service provides ML-based rightsizing recommendations?

- A) AWS Trusted Advisor
- B) AWS Cost Explorer with rightsizing recommendations
- C) AWS Compute Optimizer
- D) AWS Systems Manager Inventory

**Answer: C**
> AWS Compute Optimizer uses machine learning to analyze CloudWatch metrics and provides specific rightsizing recommendations for EC2, EBS, Lambda, and ECS on Fargate. Cost Explorer also has rightsizing but is less granular. Trusted Advisor provides basic checks but not ML-based recommendations.

---

**Score: __ / 28**
