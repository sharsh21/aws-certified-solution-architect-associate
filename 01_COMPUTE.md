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

### Real-World Use Cases & When to Choose EC2

> **The problem it solves:** You need full OS-level control over compute — custom kernel, specific hardware, persistent processes, or compliance requirements that exclude serverless.

**Choose EC2 when:**
- Your app needs persistent processes (game server, long-running daemon)
- You have strict licensing (Oracle, SQL Server BYOL) tied to physical cores/sockets
- You need GPU access (ML training, video rendering)
- You're lifting-and-shifting a legacy app that can't be refactored

**Real-World Scenarios:**
| Business Problem | EC2 Solution |
|-----------------|-------------|
| E-commerce site needs 10x capacity on Black Friday | ASG with Target Tracking (CPU 60%) + On-Demand base + Spot Instances for surge |
| Legacy Windows .NET app migration to cloud | Lift-and-shift to EC2 Windows Server — same OS, same config |
| Oracle DB with per-socket licensing | Dedicated Host — you control which physical sockets run Oracle |
| Genomics institute processes DNA sequences overnight | Spot Instances with checkpointing — 90% cost savings on fault-tolerant batch jobs |
| Financial firm needs sub-millisecond inter-node latency | Cluster Placement Group + c5n instances with enhanced networking |
| Dev team wastes money on idle dev environments | Scheduled Scaling: scale down to 0 at night, scale up at 8 AM |

**Avoid EC2 when:** Tasks run < 15 minutes → Lambda. Containerized microservices → ECS/Fargate. No infrastructure team → Elastic Beanstalk or Lambda.

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

### Real-World Use Cases & When to Choose Each Load Balancer

> **The problem it solves:** Distribute traffic across multiple targets for HA and scale, and intelligently route requests based on content.

**Real-World Scenarios:**
| Business Problem | Right Load Balancer |
|-----------------|-------------------|
| SaaS app routes `/api/*` to backend and `/` to frontend | **ALB** — path-based routing to different target groups |
| Multi-tenant platform: `customer1.app.com`, `customer2.app.com` | **ALB** — host-based routing per tenant |
| Gaming company needs static IP whitelisted in corporate firewalls | **NLB** — static Elastic IP per AZ, whitelist once |
| Real-time multiplayer game (UDP protocol) | **NLB** — supports UDP, ultra-low latency |
| Telecom company deploys 3rd-party IDS/IPS inline for all traffic | **GLB** — sends traffic through appliance before forwarding |
| Healthcare portal serving multiple microservices with WAF | **ALB** — native WAF integration, Lambda targets, rich routing |

**Exam decision rule:** HTTP/HTTPS routing intelligence → ALB. Static IP, TCP/UDP, raw performance → NLB. 3rd-party security appliance → GLB.

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

### Real-World Use Cases & When to Choose Lambda

> **The problem it solves:** Run event-driven code without provisioning or managing servers — you pay only for actual execution time (per millisecond).

**Choose Lambda when:**
- Workload is event-driven and sporadic (file uploads, API calls, scheduled jobs)
- Tasks complete in under 15 minutes
- You want zero server management and automatic scaling to zero

**Real-World Scenarios:**
| Business Problem | Lambda Solution |
|-----------------|----------------|
| Profile photos uploaded to S3 need thumbnails generated | S3 `ObjectCreated` event → Lambda resizes image → stores back to S3 |
| Startup wants a REST API with zero server management | API Gateway → Lambda → DynamoDB (fully serverless stack, scales from 0 to millions) |
| Nightly report sent every day at 6 AM | EventBridge scheduled rule (cron) → Lambda → generates report → SES email |
| Order placed in DynamoDB triggers inventory update | DynamoDB Streams → Lambda → calls inventory microservice |
| SaaS app authenticates users via custom JWT | Lambda Authorizer on API Gateway — validates JWT before every request |
| IoT sensors send millions of readings | Kinesis Data Streams → Lambda → writes to DynamoDB (auto-scales with stream) |

**Avoid Lambda when:** Task > 15 min (→ ECS/Batch). Need persistent TCP connections. Need GPU. Need shared filesystem across invocations (→ ECS + EFS).

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

### Real-World Use Cases & When to Choose ECS

> **The problem it solves:** Run containerized applications without learning Kubernetes — AWS-native orchestration that integrates deeply with IAM, ALB, CloudWatch, and Secrets Manager.

**Choose ECS (Fargate) when:** Team uses Docker but doesn't need Kubernetes. Variable workloads. Want zero infra management.
**Choose ECS (EC2) when:** Need GPU instances (ML inference). Have Reserved Instances to use. Need fine-grained placement control.

**Real-World Scenarios:**
| Business Problem | ECS Solution |
|-----------------|-------------|
| Startup has 10 microservices in Docker — no Kubernetes expertise | ECS Fargate — deploy containers, ALB routes traffic, auto-scales, zero EC2 management |
| Company migrates monolith to microservices gradually | ECS Services with ALB weighted target groups — shift traffic incrementally |
| ML inference API — GPU needed, cost is key | ECS EC2 launch type with g4dn instances + Reserved Instances for savings |
| Video transcoding pipeline — variable job length (5–60 min) | ECS Tasks triggered by SQS — each message spins up a Fargate task |
| Multi-environment CI/CD (dev/staging/prod) | Separate ECS clusters per env, ECR image promotion pipeline |

---

## EKS — Elastic Kubernetes Service

- Managed Kubernetes control plane
- Node types: EC2 (managed/self-managed node groups) or Fargate
- **EKS Anywhere:** Run EKS on-premises
- **EKS Distro:** Kubernetes distribution used by EKS
- Use when: team knows Kubernetes, need portability, open-source tooling

> **Exam Tip:** ECS = AWS-native, simpler. EKS = Kubernetes, portability, complex. Both support EC2 + Fargate.

### Real-World Use Cases & When to Choose EKS

> **The problem it solves:** Run Kubernetes on AWS when your team is already invested in the Kubernetes ecosystem — Helm charts, kubectl, service meshes (Istio), open-source tooling.

**Choose EKS when:**
- Your team already uses Kubernetes on-premises and wants a consistent experience
- You need to run the same workloads across AWS, GCP, Azure, or on-premises
- You use Kubernetes-native tooling (Helm, Istio, Prometheus, ArgoCD)

**Real-World Scenarios:**
| Business Problem | EKS Solution |
|-----------------|-------------|
| Bank runs Kubernetes on-premises and migrates to AWS | EKS — same kubectl commands, same Helm charts, minimal retraining |
| Company needs multi-cloud strategy (AWS + Azure) | EKS + AKS with same manifests — cloud portability |
| Platform team builds internal developer platform | EKS with ArgoCD (GitOps), Istio service mesh, Prometheus monitoring |
| Regulated firm needs to run some workloads on-premises | EKS Anywhere — same EKS control plane, runs on-prem servers |

**Avoid EKS when:** Team is new to containers → ECS Fargate is far simpler. Small team, fewer than 20 services → ECS overhead is lower.

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

### Real-World Use Cases & When to Choose Elastic Beanstalk

> **The problem it solves:** Deploy web applications without managing the underlying infrastructure — upload code, Beanstalk handles load balancers, auto scaling, OS patching, and monitoring.

**Choose Elastic Beanstalk when:**
- Developers want to focus on code, not infra (startup teams, small orgs)
- Migrating existing web apps with standard stacks (Java, Node.js, Python, PHP)
- You need a quick proof-of-concept with production-like infrastructure

**Real-World Scenarios:**
| Business Problem | Elastic Beanstalk Solution |
|-----------------|---------------------------|
| Dev team needs to deploy a Node.js REST API fast with zero DevOps knowledge | Upload zip → Beanstalk creates ALB, ASG, EC2, CloudWatch automatically |
| Company wants zero-downtime deploys for a PHP e-commerce app | Blue/Green deployment — swap environment URLs with no downtime |
| Legacy Java WAR file needs to move to cloud quickly | Upload WAR, Beanstalk manages Tomcat server and infrastructure |
| Startup needs staging and production environments | Two Beanstalk environments — same config, different tiers (single vs multi-instance) |

**Avoid when:** Need full infra control → CloudFormation/CDK. Running containers → ECS. Serverless functions → Lambda.

---

## AWS Batch

- Fully managed batch computing
- Runs Docker containers on EC2 or Fargate
- **Job Queue → Job Definition → Compute Environment**
- Compute Environment: Managed (AWS handles scaling) or Unmanaged
- Supports Spot Instances for cost savings
- Use for: ML, genomics, financial risk modeling, image processing

### Real-World Use Cases & When to Choose AWS Batch

> **The problem it solves:** Run hundreds of thousands of batch computing jobs at any scale — AWS handles provisioning, job scheduling, and resource optimization automatically.

**Choose AWS Batch when:**
- Jobs are long-running (> 15 min, ruling out Lambda) but don't need a persistent server
- Jobs are embarrassingly parallel (each job is independent)
- Cost optimization via Spot Instances is critical

**Real-World Scenarios:**
| Business Problem | AWS Batch Solution |
|-----------------|-------------------|
| Biotech company processes 10,000 genome sequence files nightly | Batch jobs on Spot c5 instances — each job processes one file, scales to thousands in parallel |
| Bank runs Monte Carlo simulations for risk modeling end-of-day | Batch with Managed Compute Environment — auto-provisions EC2, runs 500 parallel simulations |
| Media company transcodes 1M video clips per day | SQS queue feeds Batch job queue — Spot Instances process each clip, 70% cost savings |
| ML team trains 50 models in parallel for hyperparameter search | Batch with GPU instances (p3) — each job trains one model variant |

**The key distinction for the exam:** Lambda = < 15 min serverless. ECS = long-running containers (services). Batch = large-scale parallel job processing (minutes to hours).

---

## AWS Outposts

- AWS infrastructure on-premises
- Full AWS APIs locally
- For: latency requirements, data residency, local processing

### Real-World Use Cases & When to Choose AWS Outposts

> **The problem it solves:** Run AWS services on your own data center hardware — same APIs, same console, same tools — for workloads that can't move to the cloud due to latency, data residency, or regulatory requirements.

**Choose AWS Outposts when:**
- Regulations require data to stay within a specific country/facility (GDPR, healthcare)
- Applications require < 1ms latency to on-premises systems (manufacturing, trading)
- Hybrid architecture needs consistent API across cloud and on-premises

**Real-World Scenarios:**
| Business Problem | Outposts Solution |
|-----------------|------------------|
| Hospital must keep patient data within its own data center (HIPAA locality requirement) | Outposts rack in hospital DC — runs RDS, EC2 locally, data never leaves |
| Factory floor needs ML inference on sensor data with < 1ms latency | Outposts runs SageMaker inference locally — no network round-trip to cloud |
| Bank must store transaction records on-premises for 7 years by regulation | Outposts with S3 on Outposts — same S3 API, physically on-prem |
| Telecom needs to process 5G network data locally before sending to cloud | Outposts at edge locations — process locally, sync aggregates to AWS region |

---

## Key Exam Traps

1. **Spot interruption** — 2-minute warning before termination. Use Spot with ASG + checkpointing
2. **Dedicated Host vs Dedicated Instance** — Host = you control placement on physical server (needed for BYOL per-socket/per-core licensing)
3. **ALB vs NLB** — NLB for static IP, gaming, TCP/UDP. ALB for HTTP routing, microservices
4. **Lambda 15-min limit** — if task > 15 min, use ECS/Batch/EC2
5. **Fargate = no SSH access** — cannot access underlying host
6. **Reserved Concurrency = 0** → throttles all Lambda invocations (used to disable)
