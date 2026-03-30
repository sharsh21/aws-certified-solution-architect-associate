# Compute Services — SAA-C03 Exam Guide

---

## EC2 — Elastic Compute Cloud

### Instance Types (MEMORIZE)
| Family | Optimized For | Use Cases |
|--------|--------------|-----------|
| `t3/t4g` | Burstable general | Dev/test, low traffic web |
| `m5/m6i` | General purpose | App servers, backend |
| `c5/c6i` | Compute optimized | HPC, batch, gaming, video encoding |
| `r5/r6i` | Memory optimized | In-memory DBs, SAP, real-time big data |
| `x1e/u-*` | High memory | SAP HANA, large in-memory DBs |
| `i3/i4i` | Storage optimized | NoSQL, data warehousing, Elasticsearch |
| `p3/p4` | GPU | ML training, HPC |
| `g4dn` | GPU | ML inference, video transcoding |

### Purchasing Options (CRITICAL)
| Option | Best For | Discount |
|--------|---------|---------|
| **On-Demand** | Short-term, unpredictable | Baseline |
| **Reserved (1yr/3yr)** | Steady-state, predictable | Up to 72% |
| **Savings Plans** | Flexible reserved (compute/instance) | Up to 66% |
| **Spot Instances** | Fault-tolerant, flexible, batch | Up to 90% |
| **Dedicated Host** | Compliance, BYOL, per-socket licensing | |
| **Dedicated Instance** | Hardware isolation, no host control | |

> **Exam Tip:** Spot = cheapest but can be interrupted. Dedicated Host = you control the physical server. Dedicated Instance = isolated hardware but AWS manages the host.

### Auto Scaling Group (ASG)
- **Launch Template** (preferred) vs Launch Configuration (legacy)
- **Scaling Policies:**
  - Target Tracking — maintain metric (e.g., CPU 60%)
  - Step Scaling — respond to CloudWatch alarms
  - Scheduled Scaling — known traffic patterns
  - Predictive Scaling — ML-based forecast
- **Health Checks:** EC2 (default) or ELB health checks
- **Cooldown Period:** Default 300s — prevents rapid scale in/out
- **Lifecycle Hooks:** Run scripts before instance launch/terminate

### Placement Groups
| Type | Benefit | Use Case |
|------|---------|---------|
| **Cluster** | Low latency, high throughput | HPC, tightly coupled |
| **Spread** | Distinct hardware, max 7/AZ | Critical instances, HA |
| **Partition** | Isolated racks, up to 7 partitions | Hadoop, Cassandra, Kafka |

### Storage for EC2
- **Instance Store:** Ephemeral, highest IOPS, lost on stop/terminate
- **EBS:** Persistent block storage (see Storage guide)
- **EFS:** Shared NFS, multi-instance, Linux only

### AMI
- Region-specific (copy AMI for cross-region)
- Can be shared across accounts
- Encryption: Encrypted AMI → Encrypted volumes only
- Backed by EBS (most) or Instance Store

### EC2 Hibernation
- RAM saved to EBS, faster startup
- Requires EBS encryption, instance must be EBS-backed
- Max 60 days of hibernation

---

## ELB — Elastic Load Balancer

### Load Balancer Types (HIGH EXAM WEIGHT)
| Type | Layer | Protocol | Key Features |
|------|-------|----------|-------------|
| **ALB** (Application) | L7 | HTTP/HTTPS/gRPC | Path/host routing, WAF integration, Lambda targets |
| **NLB** (Network) | L4 | TCP/UDP/TLS | Ultra-low latency, static IP, Elastic IP support |
| **GLB** (Gateway) | L3+L4 | GENEVE | 3rd-party appliances, inline traffic inspection |
| **CLB** (Classic) | L4/L7 | Legacy | Avoid — use ALB/NLB |

### ALB Key Concepts
- **Target Groups:** EC2 instances, IP addresses, Lambda functions, ALB
- **Routing Rules:** Host-based (`api.example.com`), path-based (`/api/*`), header/query
- **Sticky Sessions:** Cookie-based (AWSALB cookie)
- **Connection Draining (Deregistration Delay):** Default 300s
- **Access Logs → S3**
- **Cross-Zone Load Balancing:** Enabled by default on ALB

### NLB Key Concepts
- Preserves client IP
- Supports static/Elastic IPs per AZ
- Ultra-low latency for gaming, IoT
- Can handle millions of requests/second
- TLS termination supported
- Cross-Zone LB: disabled by default (costs extra)

### SSL/TLS
- Certificates via ACM (AWS Certificate Manager)
- **SNI (Server Name Indication):** Multiple certs on one ALB/NLB
- Classic LB: one cert only

---

## Lambda

### Key Concepts
- Max execution: **15 minutes**
- Memory: 128MB – 10,240MB (CPU scales with memory)
- Ephemeral storage: /tmp up to **10GB**
- Deployment package: 50MB zipped, 250MB unzipped
- Container image support: up to **10GB**
- Concurrency: **1000 default** (soft limit, can increase)
- **Reserved Concurrency:** Guarantees max capacity, throttles above limit
- **Provisioned Concurrency:** Pre-warms instances, eliminates cold starts

### Lambda Triggers (Common Exam Scenarios)
- S3 events (ObjectCreated, ObjectRemoved)
- DynamoDB Streams
- Kinesis Data Streams
- SQS (event source mapping)
- SNS, EventBridge, API Gateway
- ALB (target group)
- Cognito, IoT

### Lambda@Edge vs CloudFront Functions
| | Lambda@Edge | CloudFront Functions |
|--|-------------|---------------------|
| Runtime | Node.js, Python | JavaScript only |
| Max duration | 5-30 sec | < 1ms |
| Memory | 128MB-10GB | 2MB |
| Triggers | Viewer/Origin req/res | Viewer req/res only |
| Use Case | Complex logic, auth | Header manipulation, URL rewrite |

### Cold Start Mitigation
- Provisioned Concurrency
- Keep functions warm (SnapStart for Java)
- Use ARM/Graviton2 (faster cold start, cheaper)

---

## ECS — Elastic Container Service

### Launch Types
| | EC2 Launch Type | Fargate |
|-|----------------|---------|
| Infrastructure | You manage EC2 | Serverless, AWS manages |
| Pricing | Pay for EC2 instances | Pay per vCPU/memory used |
| Control | Full control | Limited |
| Use Case | Cost-sensitive, GPU workloads | Simplicity, variable workloads |

### Key Concepts
- **Task Definition:** Blueprint — image, CPU, memory, port mappings, IAM role
- **Task:** Running instance of a task definition
- **Service:** Manages desired count, integrates with ALB, auto-scaling
- **Cluster:** Logical grouping of tasks/services
- **ECS Task Role:** IAM permissions for containers (not the EC2 instance role)
- **ECS Service Auto Scaling:** Target tracking, step, scheduled

### Networking Modes
- `awsvpc` — each task gets its own ENI (recommended, required for Fargate)
- `bridge` — Docker bridge network (legacy EC2)
- `host` — shares host network

---

## EKS — Elastic Kubernetes Service

- Managed Kubernetes control plane
- Node types: EC2 (managed/self-managed node groups) or Fargate
- **EKS Anywhere:** Run EKS on-premises
- **EKS Distro:** Kubernetes distribution used by EKS
- Use when: team knows Kubernetes, need portability, open-source tooling

> **Exam Tip:** ECS = AWS-native, simpler. EKS = Kubernetes, portability, complex. Both support EC2 + Fargate.

---

## Elastic Beanstalk

- PaaS — upload code, AWS handles infrastructure
- Supports: Java, .NET, PHP, Node.js, Python, Ruby, Go, Docker
- **Deployment Policies:**
  - All at once — fastest, has downtime
  - Rolling — no downtime, reduced capacity during deploy
  - Rolling with additional batch — no capacity loss
  - Immutable — safest, new instances, expensive
  - Blue/Green — swap environment URLs, zero downtime
- You still have access to underlying EC2, RDS, etc.
- `.ebextensions/` — customize environment configuration

---

## AWS Batch

- Fully managed batch computing
- Runs Docker containers on EC2 or Fargate
- **Job Queue → Job Definition → Compute Environment**
- Compute Environment: Managed (AWS handles scaling) or Unmanaged
- Supports Spot Instances for cost savings
- Use for: ML, genomics, financial risk modeling, image processing

---

## AWS Outposts

- AWS infrastructure on-premises
- Full AWS APIs locally
- For: latency requirements, data residency, local processing

---

## Key Exam Traps

1. **Spot interruption** — 2-minute warning before termination. Use Spot with ASG + checkpointing
2. **Dedicated Host vs Dedicated Instance** — Host = you control placement on physical server (needed for BYOL per-socket/per-core licensing)
3. **ALB vs NLB** — NLB for static IP, gaming, TCP/UDP. ALB for HTTP routing, microservices
4. **Lambda 15-min limit** — if task > 15 min, use ECS/Batch/EC2
5. **Fargate = no SSH access** — cannot access underlying host
6. **Reserved Concurrency = 0** → throttles all Lambda invocations (used to disable)
