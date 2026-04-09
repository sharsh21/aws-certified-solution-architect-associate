# Security Services — SAA-C03 Exam Guide

---

## IAM — Identity and Access Management (HIGHEST EXAM WEIGHT)

### Core Concepts
- **Users:** Individual people or applications
- **Groups:** Collection of users (cannot nest groups)
- **Roles:** Assumed by services, users, or cross-account (no permanent credentials)
- **Policies:** JSON documents defining permissions

### Policy Types
| Type | Scope | Description |
|------|-------|-------------|
| **Identity-based** | User/Group/Role | Permissions for who can do what |
| **Resource-based** | Resource (S3, KMS, etc.) | Who can access this resource |
| **Permission Boundaries** | User/Role | Max permissions (ceiling) |
| **SCPs (Service Control Policies)** | Account/OU in Organizations | Guardrails across accounts |
| **Session Policies** | AssumeRole sessions | Further restrict session |

### Policy Evaluation Logic
1. **Explicit Deny** → DENY (always wins)
2. **SCP** → if in Organizations, must allow
3. **Resource-based policy** → may grant access
4. **Identity-based policy** → must allow
5. **Permission boundary** → must allow
6. **Default** → DENY

> **Exam Tip:** Explicit Deny ALWAYS wins. If no explicit allow exists → implicit deny (access denied).

### IAM Roles — Key Patterns
- **EC2 Instance Role:** Attach role to EC2 (use instead of hardcoded credentials)
- **Cross-Account Role:** Role in Account B, trusted by Account A — user assumes role
- **Service Role:** Allow AWS service (Lambda, ECS) to call other AWS services
- **Web Identity Federation:** Mobile/web apps using Cognito or external IdP
- **SAML 2.0 Federation:** Enterprise SSO with on-premises IdP (ADFS, etc.)

### STS — Security Token Service
- `AssumeRole` — cross-account, service roles
- `AssumeRoleWithSAML` — SAML federation
- `AssumeRoleWithWebIdentity` — web/mobile (use Cognito instead)
- `GetSessionToken` — MFA enforcement

### IAM Best Practices
- Root account: enable MFA, don't use for daily tasks
- Least privilege principle
- Rotate credentials
- Use roles for EC2/Lambda (never hardcode keys)
- Enable IAM Credential Report for auditing

### Real-World Use Cases & When IAM Patterns Apply

> **The problem it solves:** Centralized, fine-grained access control for every AWS action — who (identity) can do what (action) on which resource, under what conditions.

**Real-World Scenarios:**
| Business Problem | IAM Solution |
|-----------------|-------------|
| EC2 app needs to read from S3 — developer wants to hardcode access keys | **EC2 Instance Role** — attach IAM role to EC2, credentials auto-rotated, no hardcoded keys |
| Company A wants Company B's team to access their AWS account | **Cross-Account Role** — Company A creates role trusting Company B's account; their users assume it |
| Enterprise uses Azure AD for SSO — wants same login for AWS Console | **SAML 2.0 Federation** — ADFS/Azure AD → STS AssumeRoleWithSAML → AWS Console |
| Mobile app users need to read their own S3 files | **Cognito Identity Pools** → IAM role scoped per user with `${cognito-identity.amazonaws.com:sub}` |
| Security team wants to ensure no one in entire org can disable CloudTrail | **SCP on Organizations root OU** — deny `cloudtrail:StopLogging` for all member accounts |
| Junior dev accidentally deleted production S3 bucket | **Permission Boundary** on dev IAM role — limits max permissions even if policy grants more |
| Contractor needs temporary access for 8 hours only | STS `AssumeRole` with `MaxSessionDuration` + time-based condition in policy |

**The policy evaluation mental model:** Explicit Deny beats everything → SCP limits the ceiling → resource policy can grant cross-account → identity policy must allow → permission boundary caps the max.

---

## KMS — Key Management Service

### Key Types
| Type | Who Manages | Rotation | Use Case |
|------|------------|---------|---------|
| **AWS Managed Keys** | AWS | Auto (annual) | Default for most services |
| **Customer Managed Keys (CMK)** | Customer | Optional (annual) | Full control, cross-account |
| **Customer Provided Keys** | Customer | Customer | Import own key material |

### KMS Key Operations
- **Encrypt:** plaintext → ciphertext (max 4KB)
- **Decrypt:** ciphertext → plaintext
- **GenerateDataKey:** Returns plaintext + encrypted data key (envelope encryption)
- **GenerateDataKeyWithoutPlaintext:** Returns encrypted key only

### Envelope Encryption
1. KMS generates a **Data Encryption Key (DEK)**
2. DEK encrypts your data locally (fast, no size limit)
3. Encrypted DEK stored with data
4. To decrypt: call KMS to decrypt DEK, then use DEK to decrypt data

> **Exam Tip:** Envelope encryption = encrypt large data using local DEK (not KMS directly). KMS only stores CMK, not DEK.

### KMS Multi-Region Keys
- Same key material in multiple regions
- Can encrypt in one region, decrypt in another
- Use for: disaster recovery, global applications

### Key Policies
- Resource-based policy on KMS key
- Must allow root account of key-owning account
- Control who can use/administer the key

### KMS vs CloudHSM

| | KMS | CloudHSM |
|-|-----|---------|
| Management | AWS managed | Customer managed (exclusive HSM) |
| Keys | AWS stores | Customer stores |
| FIPS | 140-2 Level 3 | 140-2 Level 3 |
| Compliance | Less strict | PCI DSS, strict compliance |
| Multi-tenancy | Multi-tenant | Single-tenant |
| Cost | Lower | Higher |
| Integration | Native AWS | Manual (PKCS#11, JCE) |

> **Exam Tip:** CloudHSM = you need to manage your own keys on dedicated hardware. KMS = AWS manages HSM but shares it.

### Real-World Use Cases & When to Use KMS

> **The problem it solves:** Managed encryption key service — create, store, and control the keys used to encrypt your data across all AWS services, with full audit trail via CloudTrail.

**Choose KMS when:** You need encryption at rest for AWS services, want centralized key management with access control, or need audit trail of all key usage (compliance).

**Real-World Scenarios:**
| Business Problem | KMS Solution |
|-----------------|-------------|
| HIPAA compliance: all RDS data must be encrypted with customer-managed keys | Create CMK (Customer Managed Key) → enable RDS encryption at creation → key policy restricts access |
| S3 bucket stores financial records — encryption required, audit who accessed keys | **SSE-KMS** — every S3 object encrypted, every decrypt call logged in CloudTrail |
| Application encrypts 100GB files — using KMS Encrypt API directly is too slow | **Envelope Encryption** — KMS generates Data Key, app encrypts file locally, stores encrypted data key with file |
| Multi-region app: encrypt in us-east-1, decrypt in eu-west-1 | **KMS Multi-Region Key** — same key material replicated, cross-region decrypt without re-encryption |
| Payment processor needs dedicated HSM, can't share with other AWS customers | **CloudHSM** — single-tenant, FIPS 140-2 Level 3, you control the HSM (not AWS) |

**KMS vs CloudHSM for the exam:** "Dedicated hardware, exclusive HSM, you manage keys" → CloudHSM. "Managed, integrated with AWS services, audit via CloudTrail" → KMS.

---

## Cognito

### User Pools vs Identity Pools

| | User Pools | Identity Pools |
|-|-----------|---------------|
| **Purpose** | User directory, authentication | AWS credentials, authorization |
| **Returns** | JWT tokens (ID, Access, Refresh) | Temporary AWS credentials |
| **Provides** | User sign-up/sign-in | Access to AWS services |
| **Supports** | Social IdPs, SAML, OIDC | User Pools, Social IdPs, SAML |

> **Exam Tip:** User Pool = who you are (authentication). Identity Pool = what you can do in AWS (authorization).

### Cognito Architecture Flow
1. User authenticates via **User Pool** → gets JWT
2. Exchange JWT with **Identity Pool** → gets temporary IAM credentials
3. Use credentials to access AWS services (S3, DynamoDB, etc.)

### Cognito Advanced Security
- MFA, adaptive authentication
- Compromised credential detection
- Advanced security features (paid)

### Real-World Use Cases & When to Use Cognito

> **The problem it solves:** Add user authentication (sign-up, sign-in, social login, MFA) to your web/mobile app without building it from scratch — and grant those users access to AWS services via temporary credentials.

**The two-service mental model:**
- **User Pool** = your user directory + authentication → returns JWT
- **Identity Pool** = exchange that JWT for AWS IAM credentials → access AWS services directly

**Real-World Scenarios:**
| Business Problem | Cognito Solution |
|-----------------|----------------|
| Mobile app needs user sign-up/sign-in (email + social login with Google) | **User Pool** — built-in UI, Google/Facebook federation, MFA, email verification |
| App users need to upload files directly to their own S3 "folder" | **Identity Pool** → IAM role scoped with `${cognito-identity.amazonaws.com:sub}` → user can only access `s3://bucket/userId/*` |
| Enterprise SSO: employees use Okta — want same login in the customer portal | **User Pool + SAML federation** — Okta as SAML IdP, Cognito is the SP |
| User Pool (authentication) + need to call DynamoDB from the mobile app | User Pool (JWT) → **Identity Pool** (AWS credentials) → direct DynamoDB access from app |
| App requires MFA for sensitive operations (bank transfers) | User Pool with MFA enforcement — SMS or TOTP (Google Authenticator) |

**Flow to memorize:** User Pool login → get JWT → swap JWT at Identity Pool → get AWS credentials → call S3/DynamoDB/etc. directly.

---

## WAF — Web Application Firewall

- Protects against common web exploits (Layer 7)
- Deploy on: CloudFront, ALB, API Gateway, AppSync, Cognito
- **Web ACL:** Rules to allow/block/count requests
- **Rule Types:**
  - IP set rules (block specific IPs)
  - Managed rule groups (OWASP Top 10, Bot Control, Known Bad Inputs)
  - Rate-based rules (DDoS protection)
  - Regex/string matching

> **Exam Tip:** WAF = L7 protection. For DDoS (L3/L4) → use Shield. WAF can work together with Shield Advanced.

### Real-World Use Cases & When to Use WAF

> **The problem it solves:** Block malicious HTTP/HTTPS traffic at Layer 7 — SQL injection, XSS, bot attacks, credential stuffing — before it ever reaches your application.

**Real-World Scenarios:**
| Business Problem | WAF Solution |
|-----------------|-------------|
| E-commerce site hit by credential stuffing bots (10K login attempts/min) | WAF Rate-Based Rule — block IPs that make > 100 requests/5 min |
| Web app vulnerable to SQL injection in search field | WAF Managed Rule Group "SQL injection" — pre-built, turn on instantly |
| News site scraped by bots causing $10K/month in server costs | WAF Bot Control managed rule — identifies and blocks bots, allows legitimate crawlers |
| PCI DSS requirement: protect cardholder data from OWASP Top 10 | WAF with AWS Managed Rules (OWASP Top 10) on ALB — one config, compliant |
| App must block all traffic from Russia and China | WAF Geo Match rule — block by country code |
| WAF logs needed for security audit | WAF full logging to S3/CloudWatch/Kinesis Firehose |

**WAF deployment priority for the exam:** CloudFront WAF = protection at the edge (global). ALB WAF = protection per region. API Gateway WAF = protection per API.

---

## Shield

| | Standard | Advanced |
|-|---------|---------|
| Cost | Free | $3,000/month |
| Protection | L3/L4 DDoS | Enhanced DDoS + L7 |
| Response | Auto | 24/7 DDoS Response Team |
| Protects | All AWS customers | CloudFront, Route53, ELB, EC2, Global Accelerator |
| Financial protection | No | Yes (credit for DDoS costs) |
| WAF integration | No | Yes |

---

## GuardDuty

- Intelligent **threat detection** (ML-based)
- Analyzes: VPC Flow Logs, CloudTrail events, DNS logs, EKS audit logs, S3 data events
- Detects: compromised instances, cryptomining, unusual API calls, malicious IPs
- No infrastructure to manage — enable and go
- Findings → SNS or EventBridge → auto-remediation (Lambda)
- **30-day free trial**, then per-volume pricing

### Real-World Use Cases & When to Use GuardDuty

> **The problem it solves:** Continuous, ML-powered threat detection across your AWS account — finds attackers and compromised resources by analyzing API calls, network flows, and DNS — without you having to write detection rules.

**Real-World Scenarios:**
| Business Problem | GuardDuty Detection |
|-----------------|---------------------|
| EC2 instance starts mining cryptocurrency (hacked instance) | GuardDuty detects Bitcoin mining traffic patterns → `CryptoCurrency:EC2/BitcoinTool.B!DNS` finding |
| IAM credentials stolen from GitHub leak — attacker calls APIs from unusual location | GuardDuty detects `UnauthorizedAccess:IAMUser/TorIPCaller` or unusual geo |
| Hacker exfiltrates data from S3 bucket to external IP | `Exfiltration:S3/MaliciousIPCaller` finding → EventBridge → Lambda isolates the IAM user |
| EC2 instance makes DNS lookups to known C2 (Command and Control) domains | `Backdoor:EC2/C&CActivity.B!DNS` — blocks via Security Group auto-remediation |
| Security team needs compliance evidence of threat monitoring | GuardDuty findings exported to S3 for 90-day audit retention |

**The exam pattern:** GuardDuty DETECTS threats (doesn't block). Add EventBridge + Lambda for automated response (isolate EC2, revoke credentials). GuardDuty ≠ WAF (L7 blocking).

---

## Macie

- Fully managed **data security and privacy** service using ML
- Discovers and protects **sensitive data in S3** (PII, credentials, financial data)
- Classifies S3 objects automatically
- Findings → EventBridge → automated response
- Use for: GDPR, HIPAA compliance, PII data discovery

---

## Inspector

- Automated **vulnerability scanning**
- Targets: EC2 instances (via SSM agent), Container images (ECR), Lambda functions
- Checks: CVEs, network reachability, CIS benchmarks
- Outputs risk score, severity findings
- Continuous scanning — not one-time

> **Exam Tip:** Inspector = vulnerability/CVE scanning. GuardDuty = threat detection/intrusion detection. Macie = sensitive data discovery.

---

## Secrets Manager

- Store, rotate, and retrieve secrets (DB passwords, API keys)
- **Automatic rotation** via Lambda (built-in or custom)
- Native integration with RDS, Aurora, Redshift, DocumentDB
- Versioning support (AWSCURRENT, AWSPREVIOUS)
- Cross-account secret sharing
- Costs money (vs SSM Parameter Store free tier)

### Real-World Use Cases & When to Use Secrets Manager

> **The problem it solves:** Eliminate hardcoded credentials in application code and config files — centralize secrets with automatic rotation so a leaked secret is quickly invalidated and audited.

**Real-World Scenarios:**
| Business Problem | Secrets Manager Solution |
|-----------------|------------------------|
| Developer hardcodes MySQL password in code — pushed to GitHub | Secrets Manager — app calls `GetSecretValue` API at runtime, no credentials in code |
| PCI DSS audit requires DB password rotation every 90 days | Secrets Manager with auto-rotation — Lambda rotates RDS password, updates secret, zero downtime |
| Microservices architecture: 20 services need the same API key | One secret in Secrets Manager, all services reference same ARN (centralized management) |
| Multi-account org: prod secrets must not be accessible by dev teams | Secrets Manager resource policy + cross-account IAM — prod secret denies dev account |
| Lambda function connects to RDS — can't use connection pooling directly | Lambda → Secrets Manager (GetSecretValue) + **RDS Proxy** (connection pooling) |

### Secrets Manager vs SSM Parameter Store

| | Secrets Manager | SSM Parameter Store |
|-|----------------|---------------------|
| Rotation | Automatic | Manual (trigger Lambda yourself) |
| Cost | $0.40/secret/month | Free (Standard), $0.05/10K (Advanced) |
| Secret size | 64KB | 4KB (Standard), 8KB (Advanced) |
| Cross-account | Yes | Limited |
| Audit | CloudTrail | CloudTrail |
| Best for | Credentials requiring rotation | Config values, non-rotating secrets |

---

## Systems Manager (SSM)

### Key Features
- **Parameter Store:** Hierarchical key-value store (with/without KMS)
- **Session Manager:** Browser/CLI shell access to EC2 (no SSH, no bastion, no port 22)
- **Patch Manager:** Automated OS patching
- **Run Command:** Execute commands on EC2 without SSH
- **State Manager:** Maintain configuration compliance
- **Automation:** Runbooks for operational tasks
- **Inventory:** Collect metadata from EC2 (installed software, OS config)

> **Exam Tip:** Session Manager = secure EC2 access without bastion hosts or open ports. Audit via CloudTrail + S3.

---

## Certificate Manager (ACM)

- Provision, manage, and deploy SSL/TLS certificates
- **Free** for public certificates
- Auto-renewal (managed certificates)
- Integrates with: ELB, CloudFront, API Gateway, Elastic Beanstalk
- **Cannot export** private key (use Private CA / ACM PCA for that)
- Region-specific (for CloudFront, must be in us-east-1)

> **Exam Tip:** ACM certificates for CloudFront MUST be in **us-east-1**.

---

## Security Hub

- Centralized security findings from GuardDuty, Inspector, Macie, Config, Firewall Manager, etc.
- Compliance checks against CIS AWS Foundations, PCI DSS, AWS Foundational Security Best Practices
- Aggregates findings across accounts in Organizations

---

## Network Security Services

### AWS Firewall Manager
- Centrally manage WAF rules, Security Groups, Shield, Network Firewall across accounts
- Requires AWS Organizations
- Automatically applies rules to new accounts/resources

### AWS Network Firewall
- Managed network firewall for VPC
- Stateful packet inspection, IPS/IDS, URL filtering
- Deploy in each VPC or centrally via Transit Gateway

---

## Key Exam Traps

1. **Permission Boundary ≠ Deny** — it sets the maximum allowed permissions, doesn't grant anything
2. **SCP in Organizations** — doesn't apply to management account, applies to member accounts
3. **ACM certificate for CloudFront** — must be in us-east-1, regardless of where CloudFront is
4. **KMS key rotation** — changes the backing key, not the key ID/ARN (old ciphertexts still decrypt)
5. **Secrets Manager vs SSM Parameter Store** — Secrets Manager costs money, has auto-rotation
6. **GuardDuty findings ≠ blocking** — it only detects; you need Lambda/EventBridge for automated response
7. **Cross-account S3 access** — both bucket policy (resource-based) AND IAM policy required
8. **WAF doesn't protect against DDoS at L3/L4** — use Shield for that
