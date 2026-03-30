# Practice Questions — Security & Monitoring (Q59–Q84)

> Format: Question → Options → Answer → Explanation
> Difficulty: Mix of Easy / Medium / Hard (marked as E/M/H)

---

### Q59 (E) — IAM Policy Evaluation
A user has an IAM policy that allows `s3:PutObject` on all S3 buckets. A bucket policy on a specific bucket explicitly DENIES `s3:PutObject` from this user. What happens when the user tries to upload to that bucket?

- A) The upload succeeds because IAM policy allows it
- B) The upload fails because explicit Deny always overrides Allow
- C) The upload succeeds because bucket policy is lower priority than IAM policy
- D) The behavior depends on whether the user is in the same account as the bucket

**Answer: B**
> In AWS IAM policy evaluation, an explicit DENY always wins — it overrides any allow from any policy. The evaluation order is: default deny → check for explicit deny → check for allow. Since bucket policy explicitly denies the action, access is denied regardless of IAM identity policy.

---

### Q60 (M) — IAM Roles
An EC2 instance needs to write logs to S3 and read parameters from SSM Parameter Store. How should permissions be granted?

- A) Create an IAM user, attach policies, and store access keys in the EC2 instance
- B) Create an IAM Role with the required policies and attach it to the EC2 instance as an instance profile
- C) Store the credentials in environment variables on the EC2 instance
- D) Use the root account credentials on the EC2 instance

**Answer: B**
> IAM Roles (instance profiles) are the correct way to grant EC2 instances AWS permissions. The credentials are temporary, automatically rotated, and never stored on disk. Hardcoding access keys (options A, C, D) is a security anti-pattern and violation of best practices.

---

### Q61 (H) — Permission Boundaries
A team lead wants to allow developers to create IAM roles for their Lambda functions, but prevent them from creating roles with more permissions than the developers themselves have. What should be used?

- A) Service Control Policies (SCPs)
- B) IAM Permission Boundaries
- C) Resource-based policies
- D) IAM Policy Conditions

**Answer: B**
> Permission Boundaries set the maximum permissions an IAM entity can have. By setting a permission boundary on roles that developers create, you ensure the created roles cannot exceed the boundary. The developer can only grant permissions they have AND that are within the boundary. SCPs apply at the account level, not to specific IAM entities.

---

### Q62 (M) — Cross-Account Access
Company A wants to allow Company B's AWS account to read objects from Company A's S3 bucket. What is the correct configuration?

- A) Create an IAM user in Company A's account and share the credentials with Company B
- B) Add Company B's account ID to the S3 bucket policy as an allowed principal
- C) Create a VPC peering connection between both accounts
- D) Enable cross-account replication from Company A to Company B

**Answer: B**
> S3 bucket policies support cross-account access by specifying the external account ARN as the principal. Company B's IAM users/roles also need permission in their own account to access Company A's bucket. For cross-account access, you need BOTH the resource-based policy (bucket policy) AND the identity-based policy.

---

### Q63 (H) — SCP vs IAM Policy
A company uses AWS Organizations. A new member account has an Administrator IAM user with full `*/*` permissions. The Security team attaches an SCP to the account's OU that denies `ec2:RunInstances`. Can the Administrator launch EC2 instances?

- A) Yes, Administrator IAM policy overrides SCPs
- B) No, SCP denies override IAM policies in member accounts
- C) Yes, but only if the account has a separate Allow SCP
- D) No, but the root user of the member account can override the SCP

**Answer: B**
> SCPs act as guardrails — they restrict the MAXIMUM permissions available in a member account. Even if an IAM user has full `*/*` permissions, if the SCP denies `ec2:RunInstances`, no one in that account (including Administrator) can launch instances. Note: SCPs do NOT apply to the management (root) account of the Organization.

---

### Q64 (M) — KMS
A company uses KMS to encrypt data. The security team needs to ensure that when an encryption key is deleted, encrypted data becomes permanently inaccessible. How does KMS key deletion work?

- A) Key is deleted immediately when requested
- B) A mandatory 7-30 day waiting period applies before deletion; data encrypted with the key becomes inaccessible after deletion
- C) Key deletion is reversible within 90 days
- D) KMS automatically re-encrypts all data before deleting a key

**Answer: B**
> KMS enforces a mandatory waiting period of 7-30 days before a key is deleted (default: 30 days). This prevents accidental deletion. During this window, you can cancel the deletion. Once deleted, the key CANNOT be recovered, and all data encrypted with it becomes permanently inaccessible. KMS does not re-encrypt data on deletion.

---

### Q65 (H) — Envelope Encryption
An application needs to encrypt a 2GB file using KMS. The developer calls `kms:Encrypt` but gets an error saying the payload exceeds 4KB. What is the correct approach?

- A) Compress the file before calling kms:Encrypt
- B) Split the file into 4KB chunks and call kms:Encrypt on each chunk
- C) Use `kms:GenerateDataKey` to get a data key, encrypt the file locally with the data key, and store the encrypted data key with the file
- D) Use SSE-S3 instead of KMS

**Answer: C**
> KMS can only directly encrypt data up to 4KB. For larger data, use **envelope encryption**: call `GenerateDataKey` to get a plaintext + encrypted data key. Use the plaintext key to encrypt the 2GB file locally (using AES-256). Store the encrypted data key alongside the encrypted file. To decrypt, call KMS to decrypt the data key, then use it locally.

---

### Q66 (M) — WAF
A company wants to protect their ALB from SQL injection and cross-site scripting (XSS) attacks. What is the MOST efficient solution?

- A) Write custom Security Group rules to block SQL injection patterns
- B) Add AWS WAF to the ALB with the AWS Managed Rules for SQL database and Core Rule Set
- C) Use AWS Shield Advanced with DDoS protection
- D) Use AWS Inspector to scan for SQL injection vulnerabilities

**Answer: B**
> AWS WAF with AWS Managed Rules provides pre-built rule groups for common vulnerabilities including SQL injection (SQLi) and XSS. It operates at Layer 7 and inspects HTTP request body, headers, and URI. Security Groups operate at Layer 3/4 (ports/IPs) and cannot inspect HTTP content. Shield is for DDoS.

---

### Q67 (E) — GuardDuty
A security team wants to detect if any EC2 instances are communicating with known malicious IP addresses or performing cryptocurrency mining. Which service should they enable?

- A) AWS Inspector
- B) AWS Macie
- C) AWS GuardDuty
- D) AWS Config with custom rules

**Answer: C**
> GuardDuty analyzes VPC Flow Logs, DNS logs, and CloudTrail to detect threats including communication with malicious IPs and cryptocurrency mining activity. Inspector scans for CVE vulnerabilities. Macie detects sensitive data in S3. Config checks configuration compliance — none of these detect active threats.

---

### Q68 (M) — Macie vs GuardDuty vs Inspector
A healthcare company stores patient records (PII, medical records) in S3. They want to automatically identify which S3 buckets contain sensitive data to ensure proper encryption and access controls. Which service should be used?

- A) GuardDuty
- B) AWS Inspector
- C) Amazon Macie
- D) AWS Config with s3-bucket-public-read-prohibited rule

**Answer: C**
> Macie uses ML to automatically discover and classify sensitive data (PII, PHI, financial data) in S3 buckets. It alerts when sensitive data is found in buckets that may be improperly configured. GuardDuty detects threats (not data classification). Inspector scans compute for vulnerabilities, not S3.

---

### Q69 (H) — Cognito
A mobile application needs to allow users to sign in with their existing Facebook or Google accounts, then access AWS resources (DynamoDB, S3) directly from the mobile client. What is the correct Cognito configuration?

- A) Use Cognito User Pool only — it handles social login and AWS credentials
- B) Use Cognito Identity Pool only — configure social IdPs directly
- C) Use Cognito User Pool (authentication with social IdPs) → federate into Cognito Identity Pool → get temporary AWS credentials
- D) Use STS AssumeRoleWithWebIdentity directly in the mobile app

**Answer: C**
> The correct pattern: User Pool authenticates users (including via social IdPs like Facebook/Google) and returns JWT tokens. Identity Pool exchanges the JWT for temporary AWS IAM credentials using STS. The app then uses these credentials to access DynamoDB/S3 directly. User Pool alone doesn't give AWS credentials; Identity Pool alone doesn't provide user management.

---

### Q70 (M) — Secrets Manager
A company rotates their RDS database password every 30 days. Currently, developers manually update the password in the application config and restart services. They want to automate this. What is the BEST solution?

- A) Store the password in SSM Parameter Store and update it manually every 30 days
- B) Use AWS Secrets Manager with automatic rotation enabled for RDS
- C) Use AWS KMS to encrypt the password and store it in S3
- D) Use AWS Config to monitor and rotate the password

**Answer: B**
> Secrets Manager has built-in automatic rotation for RDS, Aurora, Redshift, and DocumentDB. It uses a Lambda function to rotate the secret on a schedule (e.g., every 30 days). Applications retrieve the latest secret from Secrets Manager at runtime — no config changes or restarts needed. SSM Parameter Store doesn't have automatic rotation.

---

### Q71 (H) — ACM Certificate
A company uses CloudFront to serve their website `www.example.com` from us-west-2. They want to add HTTPS. They provision an ACM certificate in us-west-2, but CloudFront rejects it. What is the issue?

- A) ACM doesn't support CloudFront distributions
- B) The ACM certificate must be provisioned in **us-east-1** (N. Virginia) for use with CloudFront
- C) The certificate needs to be provisioned in the same region as the origin server
- D) CloudFront requires a Dedicated IP SSL certificate

**Answer: B**
> CloudFront is a global service and requires ACM certificates to be in **us-east-1 (N. Virginia)** — regardless of where the origin or distribution is located. This is a very commonly tested AWS gotcha. For regional services (ALB, API Gateway), the certificate must be in the same region as the resource.

---

### Q72 (M) — CloudTrail
A company needs to detect when an EC2 security group is modified (rules changed) and send an alert to the security team. What is the MOST efficient setup?

- A) Enable VPC Flow Logs and analyze with Athena
- B) Use CloudTrail to log the `AuthorizeSecurityGroupIngress` API call → EventBridge rule → SNS notification
- C) Use GuardDuty to detect security group changes
- D) Enable AWS Config with restricted-ssh rule and set up SNS notifications

**Answer: B**
> CloudTrail captures all API calls including `AuthorizeSecurityGroupIngress` and `RevokeSecurityGroupIngress`. EventBridge can filter CloudTrail events by API name and route to SNS for immediate alerting. This is a real-time response pattern. Config detects drift but may have delay. GuardDuty doesn't alert on config changes.

---

### Q73 (E) — CloudWatch Metrics
A company wants to monitor memory utilization of their EC2 instances but cannot see memory metrics in CloudWatch. What is required?

- A) Upgrade to Business support plan for enhanced monitoring
- B) Enable Enhanced Monitoring on the EC2 instance
- C) Install and configure the CloudWatch Agent on the EC2 instance
- D) Enable Detailed Monitoring on the EC2 instance

**Answer: C**
> Memory utilization is NOT available as a default CloudWatch metric (only CPU, network, disk I/O are default). The **CloudWatch Agent** must be installed on the EC2 instance to push custom metrics including memory and disk utilization. Detailed Monitoring increases metric frequency (1 min vs 5 min) but doesn't add new metric types.

---

### Q74 (M) — CloudWatch Alarms
A company wants to trigger an Auto Scaling action only when BOTH CPU utilization exceeds 80% AND memory utilization exceeds 70% simultaneously. How should this be configured?

- A) Create a CloudWatch Alarm on CPU > 80% with ASG scaling policy
- B) Create two separate alarms and use an AND condition in ASG
- C) Create a CloudWatch Composite Alarm combining both metric alarms with AND logic
- D) Use CloudWatch Metrics Insights to combine metrics

**Answer: C**
> Composite Alarms combine multiple alarms using AND/OR logic. Create Alarm 1 (CPU > 80%), Alarm 2 (memory > 70%), then a Composite Alarm that is ALARM only when BOTH are in ALARM state. Trigger the ASG scaling action from the Composite Alarm. This prevents false positives from either metric alone.

---

### Q75 (H) — AWS Config
A company wants to ensure ALL new EC2 instances launched in their account have a specific tag (`Environment = Production`). Non-compliant instances should be automatically stopped. How should this be implemented?

- A) Use IAM policy to deny EC2 RunInstances without the tag
- B) Use AWS Config rule to detect missing tags + AWS Config Remediation with SSM Automation to stop instances
- C) Use CloudTrail + EventBridge + Lambda to check and stop instances
- D) Use AWS Organizations Tag Policy to enforce tags

**Answer: B**
> AWS Config with the `required-tags` managed rule detects instances without required tags. Config Remediation can automatically invoke SSM Automation to stop non-compliant instances. Option C would also work but requires more setup. Tag Policies in Organizations enforce tag format/values but don't stop non-compliant resources.

---

### Q76 (M) — Systems Manager
A company has 500 EC2 instances spread across multiple regions. They need to apply a security patch across all instances simultaneously with a scheduled maintenance window. What is the BEST approach?

- A) SSH into each instance and run the patch manually
- B) Create an AMI with the patch and replace all instances
- C) Use AWS Systems Manager Patch Manager with a Maintenance Window
- D) Use AWS CloudFormation to redeploy all instances with the patch

**Answer: C**
> Systems Manager Patch Manager automates patching across multiple instances and regions with defined Maintenance Windows (scheduled patching periods). It applies approved patches and reports compliance status. No SSH required — SSM Agent handles execution. This scales to thousands of instances.

---

### Q77 (H) — Organizations + SCPs
A company has 20 AWS accounts in Organizations. They want to prevent any account from creating resources outside of us-east-1 and eu-west-1 for data residency compliance. What is the MOST scalable approach?

- A) Create IAM policies in each account to restrict regions
- B) Attach an SCP to the root OU that denies all actions where `aws:RequestedRegion` is not in [us-east-1, eu-west-1]
- C) Use AWS Config rules in each account to detect out-of-region resources
- D) Use CloudTrail to audit and manually delete out-of-region resources

**Answer: B**
> SCPs with `aws:RequestedRegion` condition key can prevent all API calls outside specified regions across all accounts in one SCP attached to the root OU. This automatically applies to all current and future accounts. Managing IAM policies in 20 accounts is operationally complex and doesn't scale.

---

### Q78 (E) — Shield
A company running a public-facing website frequently experiences DDoS attacks at Layer 3 and Layer 4. They want protection without any changes to their infrastructure. What is the MINIMUM action required?

- A) Enable AWS Shield Advanced on all resources
- B) AWS Shield Standard is automatically enabled at no cost for all AWS customers
- C) Deploy AWS WAF on the ALB
- D) Enable GuardDuty and configure automated DDoS response

**Answer: B**
> AWS Shield Standard is automatically enabled for ALL AWS customers at no additional cost. It protects against the most common Layer 3/4 DDoS attacks. No action required. Shield Advanced provides additional protection (L7, 24/7 DRT, financial protection) but costs $3,000/month.

---

### Q79 (M) — SSM Parameter Store vs Secrets Manager
A company stores database connection strings, API endpoints, and database passwords in a centralized configuration store. Database passwords need automatic rotation every 90 days. Non-sensitive config values should be stored at minimum cost. What is the BEST approach?

- A) Store everything in Secrets Manager
- B) Store everything in SSM Parameter Store
- C) Store database passwords in Secrets Manager (with auto-rotation), store other config values in SSM Parameter Store
- D) Store everything in S3 with KMS encryption

**Answer: C**
> Secrets Manager is ideal for secrets requiring rotation (passwords) — it has native RDS rotation. SSM Parameter Store (Standard tier) is free for non-sensitive config values. Mixing both optimizes cost: pay for Secrets Manager only where rotation is needed, use free SSM Parameter Store for static config.

---

### Q80 (H) — VPC Security Layers
A 3-tier web application has a public ALB, private EC2 app servers, and a private RDS database. The security team wants to ensure only the ALB can reach the app servers, and only the app servers can reach RDS. How should Security Groups be configured?

- A) App server SG: allow inbound from `0.0.0.0/0` on port 80. RDS SG: allow inbound from `10.0.0.0/16`
- B) App server SG: allow inbound from ALB Security Group ID on port 80. RDS SG: allow inbound from App server Security Group ID on port 3306
- C) Use NACLs to restrict traffic between tiers with IP ranges
- D) Use a transit gateway between tiers to control traffic

**Answer: B**
> Referencing Security Group IDs (instead of CIDR ranges) in Security Group rules is a best practice for dynamic environments where IPs change. ALB SG → App SG → RDS SG creates a chain where only traffic from the upstream tier is allowed. IP-based rules are fragile when Auto Scaling changes instance IPs.

---

### Q81 (M) — CloudTrail vs CloudWatch
Which statements correctly differentiate CloudTrail and CloudWatch? (Choose TWO)

- A) CloudTrail records API calls made to AWS services; CloudWatch monitors resource metrics and logs
- B) CloudTrail can trigger Lambda based on metric thresholds; CloudWatch captures API calls
- C) CloudTrail provides audit history for compliance; CloudWatch provides operational monitoring
- D) CloudTrail monitors CPU utilization; CloudWatch monitors S3 API calls

**Answer: A and C**
> CloudTrail = API audit (who called what API, when, from where) — for security and compliance. CloudWatch = metrics, logs, alarms for operational monitoring. CloudWatch triggers Lambda via alarms, not CloudTrail. CloudTrail doesn't monitor CPU; CloudWatch doesn't capture S3 API call details (CloudTrail does).

---

### Q82 (H) — IAM Federation
A company uses Active Directory on-premises (with ADFS). Employees need to access the AWS console using their AD credentials without creating IAM users in AWS. What should be configured?

- A) AWS Directory Service with Simple AD
- B) SAML 2.0 federation using IAM Identity Provider and ADFS
- C) AWS Cognito User Pool with LDAP integration
- D) Create IAM users that mirror the AD users

**Answer: B**
> SAML 2.0 federation allows ADFS to act as an Identity Provider. Users authenticate with their AD credentials on the ADFS login page, receive a SAML assertion, and exchange it with AWS STS for temporary credentials. No IAM users needed. AWS IAM supports SAML 2.0 for SSO to the console and API.

---

### Q83 (M) — Security Hub
A company has 15 AWS accounts and uses GuardDuty, Inspector, and Macie across all accounts. The security team wants a single dashboard showing all security findings from all services and accounts. What should be used?

- A) Create a custom CloudWatch dashboard aggregating findings from all services
- B) Enable AWS Security Hub with Organizations integration
- C) Configure all services to send findings to a central S3 bucket
- D) Use AWS Config Aggregator for cross-account compliance

**Answer: B**
> Security Hub aggregates findings from GuardDuty, Inspector, Macie, Config, Firewall Manager, and other partner tools across all accounts in an organization into a single view. It also runs compliance standard checks (CIS, PCI DSS). Config Aggregator is for Config compliance only.

---

### Q84 (H) — Network Firewall
A company wants to implement deep packet inspection, intrusion detection, and stateful firewall rules for all traffic entering and leaving their VPC. What should be used?

- A) Security Groups with strict inbound/outbound rules
- B) NACLs with specific port allow/deny rules
- C) AWS Network Firewall deployed at the VPC perimeter
- D) AWS WAF deployed on the Internet Gateway

**Answer: C**
> AWS Network Firewall provides stateful inspection, intrusion prevention (IPS), domain-based filtering, and deep packet inspection — features that Security Groups and NACLs don't offer. WAF only works with HTTP/HTTPS (L7) for web apps and cannot be deployed on IGW. Network Firewall sits at the VPC edge.

---

**Score: __ / 26**
