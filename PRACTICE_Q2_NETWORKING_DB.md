# Practice Questions — Networking & Databases (Q29–Q58)

> Format: Question → Options → Answer → Explanation
> Difficulty: Mix of Easy / Medium / Hard (marked as E/M/H)

---

### Q29 (E) — Security Groups vs NACLs
An EC2 instance in a private subnet cannot reach the internet. The route table has a route to a NAT Gateway. The Security Group allows all outbound traffic. What is the MOST likely cause?

- A) The Security Group is blocking inbound return traffic
- B) The NACL on the private subnet is blocking outbound traffic or inbound return traffic
- C) The NAT Gateway is in the wrong Availability Zone
- D) The VPC has no Internet Gateway

**Answer: B**
> NACLs are stateless — you must explicitly allow both outbound traffic AND inbound return traffic (ephemeral ports 1024-65535). Security Groups are stateful, so return traffic is automatically allowed. A missing NACL outbound or inbound rule is the most likely cause.

---

### Q30 (M) — VPC Peering
A company has three VPCs: VPC-A (us-east-1), VPC-B (us-east-1), VPC-C (us-east-1). They have peering between A↔B and B↔C. VPC-A needs to communicate with VPC-C. What must be done?

- A) VPC peering is transitive — A can reach C through B automatically
- B) Create a direct VPC peering connection between VPC-A and VPC-C
- C) Use Transit Gateway to enable transitive routing
- D) Enable VPC route propagation on VPC-B

**Answer: B**
> VPC peering is non-transitive. A↔B and B↔C does NOT allow A↔C. You must create a direct peering connection between A and C. If you have many VPCs, Transit Gateway is a better solution (it does support transitive routing), but for this scenario the direct answer is a new peering connection.

---

### Q31 (H) — VPC Endpoints
A Lambda function in a private VPC subnet needs to access DynamoDB. The company wants traffic to stay within the AWS network and not traverse the internet. What is the MOST cost-effective solution?

- A) Create an Interface VPC Endpoint for DynamoDB
- B) Create a Gateway VPC Endpoint for DynamoDB
- C) Create a NAT Gateway to route traffic to DynamoDB
- D) Deploy DynamoDB in the same VPC

**Answer: B**
> Gateway VPC Endpoints are available for S3 and DynamoDB — they are FREE and route traffic through the AWS network without touching the internet. Interface endpoints for DynamoDB exist but cost money per hour + per GB. NAT Gateway also costs money and routes through the internet gateway.

---

### Q32 (M) — Route 53 Routing Policies
A company has web servers in us-east-1 and ap-southeast-1. They want users to be automatically directed to the server that provides the lowest network latency. Which Route 53 routing policy should be used?

- A) Geolocation routing
- B) Weighted routing
- C) Latency-based routing
- D) Geoproximity routing

**Answer: C**
> Latency-based routing directs users to the AWS region that provides the lowest latency based on actual network measurements. Geolocation routes based on the user's location (country/continent), not performance. Geoproximity is for fine-tuning based on geographic distance with bias adjustments.

---

### Q33 (H) — Route 53 Failover
A company has a primary website in us-east-1 and a static S3 website as a backup. Route 53 should automatically failover to S3 if the primary is unhealthy. How should this be configured?

- A) Weighted routing with 90/10 split and health checks
- B) Failover routing with health check on the primary, Alias record pointing to S3 for secondary
- C) Latency-based routing with health checks
- D) Multi-Value Answer routing with two records

**Answer: B**
> Route 53 Failover routing requires a health check on the Primary record. When primary is unhealthy, Route 53 automatically serves the Secondary record (S3 static website). Use Alias record for S3 website endpoint. This is the classic active-passive DR pattern in Route 53.

---

### Q34 (M) — CloudFront + S3
A company hosts a private S3 bucket and wants to serve content through CloudFront while ensuring users CANNOT access the S3 bucket directly. What is the correct configuration?

- A) Make the S3 bucket public and use CloudFront distribution
- B) Configure Origin Access Control (OAC) on CloudFront and update S3 bucket policy to allow only CloudFront
- C) Use a signed URL on CloudFront with a public S3 bucket
- D) Create a VPC Endpoint for S3 and restrict access to CloudFront's IP range

**Answer: B**
> Origin Access Control (OAC) is the current recommended approach. CloudFront uses OAC to sign requests to S3. The S3 bucket policy allows only the CloudFront service principal. Block all direct public S3 access. OAI (Origin Access Identity) is the legacy method — OAC is the replacement.

---

### Q35 (E) — ALB Routing
A microservices application has an API Gateway frontend. The `/users` path should route to one target group and `/orders` should route to another. Which load balancer supports this?

- A) Network Load Balancer with listener rules
- B) Classic Load Balancer with path-based routing
- C) Application Load Balancer with path-based routing rules
- D) Network Load Balancer with target group weights

**Answer: C**
> ALB supports path-based routing rules (e.g., `/users/*` → Target Group A, `/orders/*` → Target Group B). NLB is Layer 4 and has no concept of HTTP paths. CLB has very limited routing capabilities.

---

### Q36 (H) — Direct Connect
A company has a Direct Connect connection to AWS. They want to ensure traffic is encrypted in transit between on-premises and AWS. What should be added?

- A) Enable encryption on the Direct Connect connection in the console
- B) Create an IPSec VPN connection over the Direct Connect connection
- C) Use AWS Shield to encrypt Direct Connect traffic
- D) Enable TLS on the application layer only

**Answer: B**
> Direct Connect does NOT encrypt traffic by default — it's a private connection but not encrypted. To add encryption, you set up an IPSec VPN tunnel ON TOP of Direct Connect. This gives you both the dedicated bandwidth of DX and the encryption of VPN.

---

### Q37 (M) — Transit Gateway
A company has 15 VPCs across 3 AWS regions and needs full mesh connectivity between all VPCs, plus connectivity to an on-premises data center. What is the MOST scalable and manageable solution?

- A) Create peering connections between every pair of VPCs (105 connections)
- B) Use AWS Transit Gateway with inter-region peering
- C) Use VPC Sharing through AWS Resource Access Manager
- D) Configure all VPCs in a single large CIDR

**Answer: B**
> Transit Gateway is a hub-and-spoke model that connects all VPCs and on-premises to a single hub. Each VPC/VPN connects once to TGW, and TGW handles routing. For cross-region, use TGW inter-region peering. 15 VPCs in full mesh would require 105 peering connections — unmanageable.

---

### Q38 (M) — NACLs
A company wants to block all traffic from a specific IP address (203.0.113.100) from reaching any EC2 instance in a subnet. Security Groups do not have a deny rule option. How should this be done?

- A) Add an outbound Security Group rule denying the IP
- B) Add an inbound NACL rule to DENY traffic from 203.0.113.100 with a lower rule number than any ALLOW rules
- C) Use Route 53 to blackhole the IP address
- D) Configure the IGW to block the IP

**Answer: B**
> Only NACLs support explicit DENY rules. NACLs evaluate rules in ascending order by rule number — the first matching rule wins. Add a DENY rule with a low rule number (e.g., 90) to block the specific IP before any ALLOW rules are evaluated. Security Groups only support ALLOW rules.

---

### Q39 (E) — NAT Gateway
EC2 instances in private subnets across two AZs need outbound internet access. The architecture must be highly available. How should NAT Gateways be deployed?

- A) One NAT Gateway in the public subnet of one AZ
- B) One NAT Gateway in a private subnet of one AZ
- C) One NAT Gateway in the public subnet of each AZ, with each private subnet's route table pointing to its AZ's NAT Gateway
- D) One NAT Gateway shared across all AZs via a centralized route table

**Answer: C**
> NAT Gateways are AZ-specific. For HA, deploy one in each AZ's public subnet and configure each private subnet's route table to use the NAT Gateway in the same AZ. If one AZ fails, other AZs are unaffected. A single NAT Gateway is a single point of failure.

---

### Q40 (H) — RDS Multi-AZ vs Read Replica
A company runs MySQL on RDS. The business reports that read-heavy reporting queries are slowing down the primary database and affecting transactional performance. What is the BEST solution?

- A) Enable RDS Multi-AZ and redirect reporting queries to the standby instance
- B) Create an RDS Read Replica and redirect reporting queries to the replica
- C) Increase the RDS instance type
- D) Enable RDS Performance Insights and optimize queries

**Answer: B**
> Multi-AZ standby cannot serve read traffic — it's only for failover. Read Replicas are specifically designed to offload read-heavy workloads. Create one or more replicas and point reporting/analytics queries to the replica endpoint. This is the classic read scaling pattern.

---

### Q41 (M) — Aurora Global Database
A company needs a database solution with RPO of seconds and RTO of less than 1 minute for cross-region disaster recovery. The application uses MySQL. What should be used?

- A) RDS MySQL with automated cross-region snapshots
- B) RDS MySQL with a cross-region read replica
- C) Aurora Global Database
- D) DynamoDB Global Tables

**Answer: C**
> Aurora Global Database replicates across up to 5 secondary regions with replication lag < 1 second (RPO in seconds) and can be promoted to primary in < 1 minute (RTO < 1 minute). RDS snapshots have RPO of hours. DynamoDB is NoSQL — not MySQL. Cross-region RDS read replica promotion takes longer.

---

### Q42 (H) — DynamoDB Partition Key Design
A DynamoDB table stores orders. The partition key is `customer_id`. A large enterprise customer generates 70% of all traffic, causing throttling on that partition. What is the BEST solution?

- A) Enable DynamoDB Auto Scaling
- B) Switch to DynamoDB On-Demand mode
- C) Use a composite partition key by appending a random suffix (write sharding) to distribute writes
- D) Increase the RCU/WCU for the table

**Answer: C**
> This is a "hot partition" problem. The fix is write sharding — append a random number (1-N) to the partition key to distribute writes across N partitions. To read, query all N partitions and aggregate. Auto Scaling and On-Demand help with capacity, but they don't fix the hot partition distribution problem.

---

### Q43 (M) — DynamoDB GSI vs LSI
A DynamoDB table uses `user_id` as partition key and `timestamp` as sort key. The application needs to query by `email` address (not the primary key). The table is already in production with millions of items. What should be created?

- A) Local Secondary Index (LSI) on `email`
- B) Global Secondary Index (GSI) on `email`
- C) Create a new table with `email` as the partition key
- D) Use DynamoDB Streams to maintain a separate lookup table

**Answer: B**
> LSIs can only be created at table creation time and must use the same partition key. Since the table already exists AND `email` is a different attribute (not the same partition key), a GSI is required. GSIs can be added to existing tables and support different partition keys.

---

### Q44 (E) — ElastiCache Selection
An application needs a caching layer that supports sorted sets for a real-time leaderboard, persistent storage in case of node failure, and high availability with automatic failover. Which ElastiCache engine should be used?

- A) Memcached
- B) Redis

**Answer: B**
> Redis supports sorted sets (required for leaderboards), data persistence (RDB/AOF snapshots), replication, and Multi-AZ automatic failover. Memcached supports none of these — it's simple key-value only, no persistence, no replication.

---

### Q45 (M) — RDS Proxy
A serverless application uses 500 Lambda functions that all connect to an RDS PostgreSQL database. The database frequently runs out of connections. What is the BEST solution?

- A) Increase the max_connections parameter on RDS
- B) Use connection pooling at the application layer with pg-pool
- C) Use Amazon RDS Proxy
- D) Switch to DynamoDB

**Answer: C**
> RDS Proxy is the AWS-native solution specifically for this problem. Lambda functions create and close connections rapidly, exhausting the database connection pool. RDS Proxy maintains a pool of warm connections to RDS and multiplexes thousands of Lambda connections into a smaller pool.

---

### Q46 (H) — DynamoDB DAX
An application performs thousands of read operations per second on DynamoDB. The team adds DAX (DynamoDB Accelerator) to reduce latency. However, a critical requirement states that some read operations must always return the most up-to-date data. What is the issue?

- A) DAX doesn't integrate with the existing application
- B) DAX doesn't support strongly consistent reads — it returns cached (eventually consistent) data
- C) DAX increases latency for write operations
- D) DAX requires changing the DynamoDB partition key

**Answer: B**
> DAX is an eventually consistent cache. It does NOT support strongly consistent reads. For operations requiring strongly consistent reads, the application must bypass DAX and read directly from DynamoDB. DAX is only beneficial for eventually consistent read-heavy workloads.

---

### Q47 (M) — Redshift
A company needs to run complex analytical queries on petabytes of historical sales data and generate BI reports. The data is stored in S3. They don't want to load all data into a database. What service should they use?

- A) RDS Aurora with read replicas
- B) Amazon Redshift with Redshift Spectrum
- C) DynamoDB with Global Tables
- D) Athena for all queries

**Answer: B**
> Redshift Spectrum allows querying S3 data directly from Redshift without loading it into the cluster. This is perfect for large historical datasets in S3 combined with hot data in Redshift. Athena is serverless SQL on S3 but doesn't provide the full Redshift analytics engine or BI integrations.

---

### Q48 (E) — Database Migration
A company runs Oracle on-premises and wants to migrate to Amazon Aurora PostgreSQL. The migration must have minimal downtime. Which services should be used?

- A) AWS DataSync + manual schema conversion
- B) AWS DMS with AWS SCT
- C) S3 export + Athena import
- D) AWS Snowball + RDS import

**Answer: B**
> DMS (Database Migration Service) migrates data with minimal downtime using CDC (Change Data Capture) to keep source and target in sync. SCT (Schema Conversion Tool) converts Oracle DDL, stored procedures, and functions to PostgreSQL syntax. This is the standard heterogeneous migration path.

---

### Q49 (H) — Route 53 CNAME vs Alias
A company wants to point their root domain `example.com` (zone apex) to an Application Load Balancer. What type of Route 53 record should be used?

- A) CNAME record pointing to the ALB DNS name
- B) A record with the ALB's IP address
- C) Alias record pointing to the ALB DNS name
- D) NS record pointing to the ALB

**Answer: C**
> CNAME records cannot be used for the zone apex (root domain). Alias records are AWS-specific, work at the zone apex, and can point to AWS resources (ELB, CloudFront, S3, API Gateway). ALB IPs change over time, so hardcoding an A record is wrong. Alias records are also free to query.

---

### Q50 (M) — CloudFront Signed URLs vs Signed Cookies
A streaming video platform needs to restrict access to premium video content on CloudFront. Each user should only be able to access videos they have subscribed to, and access should expire after 24 hours. Users typically watch multiple videos per session. What should be used?

- A) Signed URLs — one per video
- B) Signed Cookies — applies to multiple files in the session
- C) CloudFront Geo Restriction
- D) S3 Bucket Policies

**Answer: B**
> Signed Cookies are better when a user needs access to multiple files (multiple videos in a session) — you set one cookie that grants access to all matching content. Signed URLs are for single-object access. Since users watch multiple videos per session, Signed Cookies eliminates the need to generate a new URL for each video.

---

### Q51 (H) — API Gateway
A mobile application uses API Gateway + Lambda. The API is getting hit with 50,000 requests per second from automated bots. The company wants to block the bots while allowing legitimate users. What is the BEST solution?

- A) Enable API Gateway throttling at the stage level
- B) Attach an AWS WAF Web ACL with Bot Control managed rule group to the API Gateway
- C) Use Route 53 Latency routing to distribute load
- D) Enable API Gateway caching

**Answer: B**
> WAF Bot Control is specifically designed to detect and block bot traffic using managed rule groups. It can identify common bots, scrapers, and crawlers. API Gateway throttling rate-limits all traffic including legitimate users. WAF + Bot Control is the surgical solution that blocks bots while allowing real users.

---

### Q52 (M) — VPC Flow Logs
A security team needs to monitor all network traffic in a VPC for compliance. They want to store the logs in S3 for long-term retention and analyze them with Athena. What should be configured?

- A) Enable VPC Flow Logs at the VPC level, send to CloudWatch Logs, and export to S3
- B) Enable VPC Flow Logs at the VPC level, send directly to S3
- C) Install network monitoring agents on all EC2 instances
- D) Use AWS CloudTrail to capture network traffic

**Answer: B**
> VPC Flow Logs can be sent directly to S3, which is simpler and more cost-effective than going through CloudWatch Logs first. Once in S3, Athena can query them directly. CloudTrail captures API calls, not network traffic. Agents on every instance is operationally complex.

---

### Q53 (E) — Bastion Host Alternative
A company has EC2 instances in private subnets with no public IPs. The security team wants to allow engineers to access these instances for troubleshooting without opening port 22 or deploying bastion hosts. What is the recommended solution?

- A) Add a NAT Gateway to allow SSH from on-premises
- B) Use AWS Systems Manager Session Manager
- C) Deploy a bastion host in the public subnet
- D) Enable EC2 Instance Connect

**Answer: B**
> SSM Session Manager provides secure shell access to EC2 without opening any inbound ports, without a bastion host, and without managing SSH keys. Access is controlled by IAM policies, and all sessions are logged to CloudWatch/S3. EC2 Instance Connect still requires port 22 open.

---

### Q54 (H) — Global Accelerator vs CloudFront
A gaming company has game servers in us-east-1 and ap-northeast-1. They need to route players to the nearest server with failover capability, using static IP addresses that can be whitelisted by corporate firewalls. The traffic is TCP/UDP. What should be used?

- A) CloudFront with Lambda@Edge for routing
- B) Route 53 with Latency-based routing
- C) AWS Global Accelerator
- D) NLB with cross-zone load balancing

**Answer: C**
> Global Accelerator provides 2 static Anycast IPs that route TCP/UDP traffic over the AWS global network to the nearest healthy endpoint. It supports failover and is ideal for gaming (TCP/UDP). CloudFront is for HTTP content caching, not TCP/UDP gaming traffic. Static IPs are a key requirement met only by Global Accelerator.

---

### Q55 (M) — Aurora Serverless
A startup has a development database that is only accessed during business hours (8 hours/day). The database should scale to zero when not in use to minimize costs. What should be used?

- A) RDS MySQL with scheduled stop/start using Lambda
- B) Aurora Serverless v2
- C) Aurora Serverless v1
- D) DynamoDB with On-Demand capacity

**Answer: C**
> Aurora Serverless v1 can scale to zero (pause when inactive) and resume automatically on the next request. Aurora Serverless v2 does NOT scale to zero — it has a minimum ACU of 0.5. For development environments where zero cost during idle is important, Serverless v1 is the correct choice.

---

### Q56 (H) — DynamoDB Streams + Lambda
A company wants to send a welcome email whenever a new user is inserted into a DynamoDB table. How should this be implemented with the LEAST operational overhead?

- A) Poll DynamoDB every minute with a Lambda function using Scan
- B) Enable DynamoDB Streams and configure Lambda as a trigger on the stream
- C) Use EventBridge Scheduled Rule to check for new users
- D) Use SQS to queue new user events and trigger Lambda

**Answer: B**
> DynamoDB Streams captures item-level changes in real time. Lambda can be configured as a trigger (event source mapping) that fires on every new record in the stream. This is fully serverless, event-driven, and has zero operational overhead. Polling with Scan is inefficient and costly.

---

### Q57 (M) — Neptune
A social media company needs to query complex relationships like "friends of friends" or "people you may know" — queries that require traversing multiple relationship levels. Which database is MOST appropriate?

- A) Amazon RDS PostgreSQL with recursive CTEs
- B) Amazon DynamoDB with GSI
- C) Amazon Neptune
- D) Amazon DocumentDB

**Answer: C**
> Neptune is a purpose-built graph database optimized for relationship traversal. Graph queries like "friends of friends" that would require complex multi-join SQL queries in relational DBs are native to Neptune's Gremlin/SPARQL query languages. It's orders of magnitude more efficient for graph use cases.

---

### Q58 (E) — Database Selection
A startup is building a mobile app with unknown scaling requirements. The data model is simple key-value. They want a database that can handle millions of requests per second with single-digit millisecond latency at any scale without managing infrastructure. What should they use?

- A) RDS Aurora Serverless
- B) Amazon DynamoDB with On-Demand capacity
- C) ElastiCache Redis
- D) Amazon RDS MySQL with read replicas

**Answer: B**
> DynamoDB is designed for millions of requests per second at single-digit millisecond latency, fully serverless with On-Demand capacity that scales automatically. Aurora Serverless is relational (SQL), not key-value, and has connection limits. ElastiCache is a cache, not a primary database.

---

**Score: __ / 30**
