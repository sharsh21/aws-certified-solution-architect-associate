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

---

## Global Accelerator

- Network service using AWS global network
- 2 static Anycast IPs
- Routes to nearest healthy endpoint (EC2, ALB, NLB, EIP)
- **vs CloudFront:**
  - CloudFront = cache content (HTTP/HTTPS), optimizes for cacheable content
  - Global Accelerator = accelerate TCP/UDP, non-cacheable (gaming, IoT, VoIP), static IPs

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
