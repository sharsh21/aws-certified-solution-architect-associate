# Networking Services — SAA-C03 Exam Guide

---

## VPC — Virtual Private Cloud (HIGHEST EXAM WEIGHT)

### Core Components
- **VPC:** Logical isolated network (up to 5 per region, 5 CIDRs per VPC)
- **Subnet:** Partition of VPC in one AZ (public or private)
- **Internet Gateway (IGW):** Enables internet access for VPC (1 per VPC)
- **NAT Gateway:** Private subnet → internet (outbound only), managed, AZ-specific
- **NAT Instance:** Self-managed EC2, cheaper, can be used as bastion
- **Route Table:** Controls traffic routing (one per subnet)
- **CIDR:** IPv4 block size /16 (max) to /28 (min)

### Public vs Private Subnet
- **Public Subnet:** Route table has route to IGW (0.0.0.0/0 → igw-xxx)
- **Private Subnet:** No route to IGW; uses NAT Gateway for outbound internet

### Security Groups vs NACLs (CRITICAL DISTINCTION)

| | Security Group | NACL |
|-|--------------|------|
| Level | Instance (ENI) | Subnet |
| State | **Stateful** (return traffic auto allowed) | **Stateless** (must allow inbound + outbound explicitly) |
| Rules | Allow only | Allow AND Deny |
| Evaluation | All rules evaluated | Rules evaluated in order (lowest rule # first) |
| Default | Deny all inbound, allow all outbound | Allow all in/out |

> **Exam Tip:** NACL = stateless subnet firewall. SG = stateful instance firewall. NACLs use rule numbers — lower number = higher priority.

### VPC Flow Logs
- Capture IP traffic at VPC/subnet/ENI level
- Sent to CloudWatch Logs or S3
- Does NOT capture: DNS queries, DHCP, license activation, metadata traffic (169.254.x.x)

### VPC Peering
- 1:1 connection between VPCs (same or different region/account)
- **Non-transitive** — A↔B and B↔C does NOT mean A↔C
- CIDR ranges must not overlap
- Update route tables in BOTH VPCs

### VPC Endpoints (CRITICAL)
| Type | How | Use Case |
|------|-----|---------|
| **Interface Endpoint** | ENI with private IP, powered by PrivateLink | Most AWS services (SSM, SQS, SNS, etc.) |
| **Gateway Endpoint** | Route table entry | **S3 and DynamoDB only** (free) |

> **Exam Tip:** Gateway endpoints = only S3 and DynamoDB, free, route table based. Interface endpoints = everything else, cost money.

### Transit Gateway
- Hub-and-spoke model — connect thousands of VPCs + on-premises
- Transitive routing (solves VPC peering non-transitivity)
- Supports VPN and Direct Connect attachments
- Route tables control inter-VPC communication
- **Multicast** support

### VPN and Direct Connect
| | Site-to-Site VPN | Direct Connect |
|-|-----------------|----------------|
| Connection | Over internet (encrypted IPSec) | Dedicated private connection |
| Setup Time | Minutes | Weeks/months |
| Bandwidth | Up to 1.25 Gbps | 1 Gbps to 100 Gbps |
| Reliability | Less (internet dependent) | High (SLA-backed) |
| Cost | Low | Higher |

- **VPN CloudHub:** Multiple sites connect to VGW (hub-and-spoke VPN)
- **Direct Connect Gateway:** Connect to multiple VPCs across regions via one DX connection
- **DX + VPN backup:** Use VPN as backup for Direct Connect (HA pattern)

### Bastion Host / Jump Box
- EC2 in public subnet, SSH/RDP to instances in private subnet
- Security Group: allow port 22/3389 from specific IPs only
- **AWS Systems Manager Session Manager:** Alternative (no bastion needed, no open ports)

### Real-World Use Cases & When to Design with VPC

> **The problem it solves:** Network isolation in the cloud — your own logically private network where you control IP ranges, subnets, routing, and firewalls.

**VPC Design Scenarios:**
| Business Problem | VPC Solution |
|-----------------|-------------|
| Multi-tier web app (web, app, DB layers must be isolated) | 3 private subnets (one per tier) + 1 public subnet for ALB. DB layer has no route to internet |
| Dev and prod must be completely isolated but share some services | Separate VPCs per environment + VPC Peering (or Transit Gateway) for shared services VPC |
| 50 VPCs across 10 accounts need to communicate without peering mess | **Transit Gateway** — hub-and-spoke, one attachment per VPC, transitive routing |
| Lambda functions must access a private RDS without internet exposure | **VPC Interface Endpoint** (PrivateLink) or deploy Lambda in VPC private subnet |
| EC2 in private subnet needs to download packages from internet | **NAT Gateway** in public subnet — outbound only, private subnet has route 0.0.0.0/0 → NAT |
| Security team requires all S3 traffic to stay off the internet | **S3 Gateway Endpoint** — free, route-table-based, traffic stays on AWS network |
| On-premises data center must securely connect to 5 VPCs | **Direct Connect Gateway** + Transit Gateway — one DX, connects all VPCs |

**Critical VPC rules for the exam:**
- Subnets are AZ-specific — for HA, create subnets in at least 2 AZs
- NAT Gateway is AZ-specific — deploy one per AZ for HA (each AZ's private subnet routes to its local NAT)
- Security Groups = stateful (return traffic auto-allowed). NACLs = stateless (must allow both directions)
- VPC Peering = non-transitive. Transit Gateway = transitive (the hub solves this)

---

## Route 53

### Routing Policies (HIGH EXAM WEIGHT — MEMORIZE ALL)

| Policy | How It Works | Use Case |
|--------|-------------|---------|
| **Simple** | Single resource, random if multiple IPs | Single resource, no health checks |
| **Weighted** | % split between resources | A/B testing, gradual migration |
| **Latency** | Routes to lowest latency region | Global apps, performance |
| **Failover** | Active-passive, uses health checks | DR, HA |
| **Geolocation** | Based on user's location (country/continent) | Content localization, compliance |
| **Geoproximity** | Based on geographic distance + bias | Fine-tune routing by region size |
| **Multi-Value Answer** | Multiple IPs + health checks | Basic load balancing with health |
| **IP-Based** | Based on client IP/CIDR | Route specific networks |

> **Exam Tip:**
> - Latency ≠ Geolocation. Latency = performance-based, Geolocation = location-based compliance/content.
> - Multi-Value ≠ ELB. It's client-side selection from healthy records.
> - Failover requires health checks.

### Record Types
- **A:** Domain → IPv4
- **AAAA:** Domain → IPv6
- **CNAME:** Domain → another domain (cannot be used for zone apex/root domain)
- **Alias:** AWS-native, maps to AWS resources (ELB, CloudFront, S3, API GW) — FREE, works at zone apex
- **NS:** Name servers for the hosted zone
- **MX:** Mail server
- **TXT:** Verification records

> **Exam Tip:** CNAME cannot be used for root domain (example.com). Use Alias record for root domain pointing to AWS resources.

### Health Checks
- Monitor endpoints, other health checks, or CloudWatch alarms
- **Calculated health checks:** Combine multiple health checks (AND/OR logic)
- **Private resources:** Health check via CloudWatch alarm

### Route 53 Resolver
- **Resolver Inbound Endpoint:** On-premises → Route 53 DNS
- **Resolver Outbound Endpoint:** VPC → on-premises DNS
- **Forwarding Rules:** Forward specific domains to custom resolvers

### Real-World Use Cases & When to Use Each Routing Policy

> **The problem it solves:** DNS management with traffic routing intelligence — direct users to the right endpoint based on performance, geography, failover, or traffic weighting.

**Routing Policy Decision Table:**
| Scenario | Routing Policy |
|----------|---------------|
| Global SaaS app — route users to nearest AWS region for best speed | **Latency** — measures actual network latency, sends to lowest-latency region |
| A/B test a new feature on 10% of traffic | **Weighted** — 90% weight to v1, 10% weight to v2 |
| EU users must only hit EU servers (GDPR data residency) | **Geolocation** — EU traffic → eu-west-1, US traffic → us-east-1 |
| Primary region goes down — failover to DR region automatically | **Failover** — health check on primary; if unhealthy, DNS points to secondary |
| Show different content to users in different countries (media rights) | **Geolocation** — UK gets BBC content, US gets NBC content |
| Enterprise wants to gradually shift traffic from on-premises to AWS | **Weighted** — start at 10% AWS, increase weekly to 100% |
| Return multiple healthy IPs and let client pick | **Multi-Value Answer** — returns up to 8 healthy records |

**Key distinctions to memorize:**
- **Latency** = performance-based (measures real network latency to AWS regions)
- **Geolocation** = location-based (user's country/continent — for compliance/content)
- **Geoproximity** = distance + bias (use Traffic Flow to fine-tune geographic boundaries)
- **Failover** always requires health checks on the primary record

---

## CloudFront

### Key Concepts
- CDN — cache content at **Edge Locations** (400+ globally)
- **Origin:** S3, ALB, EC2, HTTP server, API Gateway
- **Distribution:** CloudFront endpoint
- **TTL:** Default cache duration
- **Cache Invalidation:** Remove cached objects (costs per path)

### CloudFront Origins
| Origin | Notes |
|--------|-------|
| S3 Bucket | OAC (Origin Access Control) for private S3 |
| S3 Static Website | Use HTTP endpoint |
| ALB | Must be public, CloudFront SG bypass via custom header |
| EC2 | Must be public |
| API Gateway | Edge-optimized API |

### Origin Access Control (OAC)
- Successor to OAI (Origin Access Identity)
- Allows CloudFront to access private S3
- S3 Bucket Policy: allow CloudFront service principal
- Block all direct S3 access

### Cache Behavior
- Path patterns (`/api/*`, `/images/*`) route to different origins
- Configure TTL per behavior
- Cache based on: headers, query strings, cookies
- **Viewer Protocol Policy:** HTTP and HTTPS, HTTPS only, Redirect HTTP to HTTPS

### Security
- **AWS WAF** integration (at CloudFront edge)
- **Shield Standard:** Auto, protects against DDoS
- **Shield Advanced:** Enhanced DDoS protection + 24/7 support
- **Geo Restriction:** Allow/deny by country
- **Signed URLs:** Single object access, expiry time, IP restriction
- **Signed Cookies:** Multiple objects, streaming

### CloudFront Functions vs Lambda@Edge
- See Compute guide for comparison

### Pricing
- **Price Classes:** All (global), 100 (NA+EU only), 200 (most regions)
- Cache hit rate → lower origin cost

### Real-World Use Cases & When to Use CloudFront

> **The problem it solves:** Cache and deliver content from 400+ global edge locations — reduces latency, cuts origin costs, and provides a security perimeter (WAF, DDoS protection) at the edge.

**Choose CloudFront when:** Static or cacheable content needs global low-latency delivery. You need DDoS protection + WAF at the edge. You want to serve private S3 content without exposing the bucket.

**Real-World Scenarios:**
| Business Problem | CloudFront Solution |
|-----------------|-------------------|
| Netflix-like platform: stream videos to users worldwide with low buffering | CloudFront distribution + S3 origin — edge caches video segments near users |
| S3 bucket hosting React app — direct S3 URL is slow for Asia users | CloudFront with S3 origin (OAC) — cached at nearest edge, S3 stays private |
| E-commerce site hit by bot DDoS during flash sale | CloudFront + WAF + Shield Advanced — DDoS absorbed at edge, blocks bot IPs |
| API is slow because every request hits the backend | CloudFront caches GET responses at edge — 80% cache hit rate → 80% less origin load |
| Content must be locked to paying subscribers only (streaming) | CloudFront Signed URLs — each video URL has expiry + user-specific signature |
| Multi-tenant SaaS: each tenant's domain needs its own TLS cert | CloudFront SNI — one distribution, multiple ACM certs, free |
| Company wants to block access from sanctioned countries | CloudFront Geo Restriction — block by country code |

**CloudFront vs Global Accelerator for the exam:**
- CloudFront = caches HTTP/HTTPS content at edge. Best for static/cacheable content.
- Global Accelerator = routes non-cacheable TCP/UDP traffic through AWS backbone. Best for gaming, VoIP, IoT.

---

## API Gateway

### Types
| Type | Protocol | Use Case |
|------|---------|---------|
| **HTTP API** | HTTP/WebSocket | Low latency, cheaper, OIDC/JWT auth |
| **REST API** | HTTP | Full features, API keys, usage plans |
| **WebSocket API** | WebSocket | Real-time bidirectional (chat, gaming) |

### Key Features
- **Stages:** dev, staging, prod
- **Stage Variables:** Like environment variables per stage
- **Canary Deployments:** Route % traffic to new version
- **Throttling:** 10,000 RPS default, 5,000 burst
- **Usage Plans + API Keys:** Rate limiting per client
- **Caching:** Cache responses at stage level (TTL 0-3600s)
- **Integration Types:** Lambda, HTTP, AWS Service, Mock

### Authentication
- IAM roles/policies
- Lambda Authorizer (custom auth, JWT, OAuth)
- Cognito User Pools

### Real-World Use Cases & When to Use API Gateway

> **The problem it solves:** Fully managed API layer that handles traffic management, authorization, monitoring, versioning, and throttling — so you don't build all of that yourself.

**API Type decision:**
- **REST API** — Full feature set, API keys, usage plans, request/response transformation, caching
- **HTTP API** — 70% cheaper, lower latency, simpler — for Lambda/HTTP backends with OIDC/JWT auth
- **WebSocket API** — Real-time bidirectional communication (chat, live dashboards, gaming)

**Real-World Scenarios:**
| Business Problem | API Gateway Solution |
|-----------------|---------------------|
| Startup wants serverless REST API (no server management) | HTTP API → Lambda → DynamoDB — fully serverless, auto-scales from 0 |
| Public API product: different clients have different rate limits | REST API + Usage Plans + API Keys — throttle per customer tier |
| Real-time chat app needs persistent connections | WebSocket API → Lambda → DynamoDB — maintains connection state |
| Backend needs to throttle at 1,000 RPS per client to prevent abuse | REST API rate-based throttling + WAF integration |
| Canary release: test new Lambda version with 5% of API traffic | REST API Canary Deployment — 95% goes to prod, 5% to new stage |
| Internal API only accessible from within VPC | Private REST API with VPC Endpoint (Interface) |

---

## Direct Connect

- Dedicated private network connection to AWS
- Speeds: 1Gbps, 10Gbps, 100Gbps (hosted: 50Mbps-10Gbps)
- **Not encrypted by default** — add VPN on top for encryption
- Use cases: large data transfer, consistent performance, compliance

### Direct Connect Gateway
- Single DX connection → multiple VPCs in different regions
- Cannot be used between two VPCs (not a transit mechanism)

### Resiliency
- Multiple DX connections: standard vs maximum resiliency
- DX + VPN as backup

### Real-World Use Cases & When to Use Direct Connect

> **The problem it solves:** A private, dedicated network pipe between your data center and AWS — consistent bandwidth, low latency, no internet variability, and potentially lower data transfer costs at scale.

**Choose Direct Connect when:** Data volume is large (TB/day+), latency must be predictable, compliance requires avoiding the public internet, or you're processing sensitive data.

**Real-World Scenarios:**
| Business Problem | Direct Connect Solution |
|-----------------|------------------------|
| Bank must transfer 50TB of trading data to AWS every night — internet is too slow/unreliable | 10 Gbps DX connection — dedicated pipe, transfers in < 2 hours nightly |
| Healthcare company sends CT scans (10GB each) to AWS for AI analysis — HIPAA requires private network | DX — data never traverses the public internet, meets HIPAA network requirements |
| Enterprise has 200ms latency issues with S3 API over internet | DX reduces latency to 10-20ms — consistent, not internet-dependent |
| Company needs DX but can't afford downtime if link fails | DX (primary) + Site-to-Site VPN (backup) — failover via BGP |
| One DX connection must reach VPCs in us-east-1, eu-west-1, ap-southeast-1 | DX + **Direct Connect Gateway** — one DX, connect to multiple VPCs across regions |

**Critical exam note:** Direct Connect is NOT encrypted by default. For encrypted DX: set up Site-to-Site VPN over the DX connection (MACsec for dedicated connections).

---

## Global Accelerator

- Network service using AWS global network
- 2 static Anycast IPs
- Routes to nearest healthy endpoint (EC2, ALB, NLB, EIP)
- **vs CloudFront:**
  - CloudFront = cache content (HTTP/HTTPS), optimizes for cacheable content
  - Global Accelerator = accelerate TCP/UDP, non-cacheable (gaming, IoT, VoIP), static IPs

### Real-World Use Cases & When to Use Global Accelerator

> **The problem it solves:** Route global user traffic through AWS's private backbone network instead of the unpredictable public internet — improving performance for non-cacheable workloads and providing 2 static IPs for whitelisting.

**Choose Global Accelerator when:**
- Application is non-HTTP (gaming: TCP/UDP, VoIP, IoT)
- You need static IP addresses globally (firewall rules, client whitelisting)
- You need instant failover between regions (health-check-based, <30 seconds)

**Real-World Scenarios:**
| Business Problem | Global Accelerator Solution |
|-----------------|---------------------------|
| Mobile game with players in 50 countries — 200ms latency is unacceptable | Global Accelerator routes players through AWS backbone from nearest PoP — cuts latency 60% |
| Enterprise app: corporate firewall must whitelist API IPs — they change with ELB | 2 static Anycast IPs from Global Accelerator — never change, whitelist once forever |
| Financial trading platform needs sub-5ms regional failover | Global Accelerator health-checks ALBs in 2 regions — instant failover via Anycast routing |
| IoT fleet (UDP protocol) spans 30 countries | Global Accelerator accelerates UDP traffic through AWS network — NLB as endpoint |

---

## Elastic IP (EIP)

- Static IPv4 address for EC2 instance
- 5 per region default (soft limit)
- Charged when NOT associated with a running instance
- Can be moved between instances for failover

---

## Key Networking Architecture Patterns

### 3-Tier Architecture
```
Internet → IGW → ALB (public subnet) → EC2 App (private subnet) → RDS (private subnet)
                                      ↑
                                  NAT Gateway (public subnet) for outbound from private
```

### Multi-Region HA
```
Route 53 Failover → CloudFront → ALB → EC2 Auto Scaling
                                      → RDS (Multi-AZ + Cross-region replica)
```

---

## Key Exam Traps

1. **NAT Gateway is AZ-specific** — deploy in each AZ for HA, add route from each AZ's private subnet
2. **VPC Peering is non-transitive** — for hub-and-spoke, use Transit Gateway
3. **Gateway Endpoint = S3 + DynamoDB only (free)**, Interface Endpoint = everything else (costs)
4. **Route 53 Alias record = free**, CNAME costs money per query
5. **CloudFront + S3** — always use OAC (not OAI), block direct S3 access
6. **Direct Connect is NOT encrypted** — use IPSec VPN over DX for encryption
7. **Global Accelerator ≠ CloudFront** — GA is for non-HTTP/TCP/UDP, static IPs
8. **NACL ephemeral ports** — allow outbound 1024-65535 for response traffic
