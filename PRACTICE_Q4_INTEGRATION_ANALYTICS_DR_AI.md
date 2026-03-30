# Practice Questions — Integration, Analytics, DR & AI/ML (Q85–Q117)

> Format: Question → Options → Answer → Explanation
> Difficulty: Mix of Easy / Medium / Hard (marked as E/M/H)

---

### Q85 (E) — SQS vs SNS
A company wants to send a notification to BOTH an email subscription and an SQS queue whenever an order is placed. Different teams process the same order event independently. What is the correct architecture?

- A) SQS queue with two separate consumers polling
- B) SNS topic with two subscriptions: one Email, one SQS queue
- C) Two separate SQS queues with the same messages duplicated by the application
- D) EventBridge with a single SQS target

**Answer: B**
> SNS fan-out: publish once to SNS topic, deliver to all subscribers simultaneously (email + SQS). Both teams receive the same event independently. SQS with two consumers would have each message consumed by only ONE consumer (competing consumers). SNS fan-out is the canonical pattern for parallel processing of the same event.

---

### Q86 (M) — SQS DLQ
A Lambda function processes messages from an SQS queue. Some messages cause processing failures and keep being requeued, blocking the queue. How should this be handled?

- A) Delete failed messages manually from the console
- B) Set the Lambda function's retry count to 0
- C) Configure a Dead Letter Queue (DLQ) on the SQS queue with a maxReceiveCount of 3
- D) Increase the visibility timeout to 24 hours

**Answer: C**
> A DLQ with `maxReceiveCount=3` means after 3 failed processing attempts, the message is moved to the DLQ. This prevents poison pill messages from blocking the queue. The DLQ can be monitored, and failed messages can be inspected/reprocessed. Visibility timeout just delays reprocessing, doesn't resolve poison pill messages.

---

### Q87 (H) — Kinesis vs SQS
A company processes financial transactions in real time. Requirements: (1) maintain ordering of transactions per account, (2) replay the last 7 days of transactions if needed, (3) multiple applications (analytics, fraud detection, audit) must read the same stream independently. Which service should be used?

- A) SQS Standard Queue
- B) SQS FIFO Queue
- C) Amazon Kinesis Data Streams
- D) Amazon SNS with SQS fan-out

**Answer: C**
> Kinesis Data Streams meets all requirements: (1) ordering guaranteed within a shard — use account_id as partition key, (2) configurable retention up to 365 days for replay, (3) multiple independent consumers (Enhanced Fan-Out). SQS FIFO can't be consumed by multiple independent consumers reading the same data, and has no replay capability.

---

### Q88 (M) — EventBridge
A company wants to trigger a Lambda function every time an EC2 instance changes state to "stopped" across any account in their AWS Organization. What is the MOST efficient setup?

- A) Enable CloudTrail in all accounts and configure a CloudWatch Alarm on StopInstances API
- B) Use EventBridge with an event pattern matching `EC2 Instance State-change Notification` where state = "stopped" on the default event bus
- C) Configure EventBridge on the management account with cross-account event bus access and create a rule for EC2 state changes
- D) Use GuardDuty to detect stopped instances

**Answer: C**
> EventBridge supports cross-account and cross-organization event routing. Set up cross-account access on the central account's event bus, allow member accounts to send events, then create one rule in the central account matching EC2 state-change (stopped) events. Lambda triggers from the central bus.

---

### Q89 (E) — Step Functions
A company runs an order processing workflow: validate order → charge payment → update inventory → send confirmation email. If any step fails, the workflow should stop and trigger a rollback. What is the BEST service?

- A) Multiple Lambda functions chained via SQS
- B) AWS Step Functions Standard Workflow
- C) EventBridge with rule chaining
- D) AWS Batch with job dependencies

**Answer: B**
> Step Functions is designed for orchestrating multi-step workflows with error handling, retries, and branching. Each step can have catch/retry logic. On failure, a fallback state can trigger compensating transactions (rollback). SQS chaining doesn't provide orchestration or error handling across steps.

---

### Q90 (H) — Kinesis Firehose
A company streams application logs from EC2 instances to be stored in S3 for analysis and also indexed in OpenSearch for real-time dashboards. The solution must require minimal operational overhead. Which architecture is BEST?

- A) Lambda function reads logs and writes to both S3 and OpenSearch
- B) Kinesis Data Firehose with delivery to S3 and a separate Firehose to OpenSearch; both fed from a Kinesis Data Stream
- C) Directly write logs to OpenSearch using the Elasticsearch API from each EC2 instance
- D) CloudWatch Logs with a subscription filter to Lambda that writes to S3 and OpenSearch

**Answer: B**
> Kinesis Data Firehose natively delivers to both S3 and OpenSearch with zero management. Use Kinesis Data Streams as the source (for decoupling), then fan out to two Firehose streams. Firehose handles batching, buffering, and retries automatically. Lambda (option A/D) adds operational overhead.

---

### Q91 (M) — Amazon MQ
A company is migrating a Java EE application from on-premises to AWS. The application uses JMS (Java Message Service) with ActiveMQ for messaging. The development team wants minimal code changes. What should be used?

- A) Amazon SQS with JMS SDK
- B) Amazon SNS with JMS adapter
- C) Amazon MQ with ActiveMQ broker
- D) Kinesis Data Streams with custom JMS wrapper

**Answer: C**
> Amazon MQ is a managed message broker for Apache ActiveMQ and RabbitMQ. It supports standard protocols including JMS, AMQP, STOMP, MQTT, and OpenWire — allowing existing applications to connect without code changes. SQS is cloud-native but requires rewriting the messaging layer.

---

### Q92 (H) — Decoupling Architecture
A company's image upload service directly calls the image processing service synchronously. When the processing service goes down, the upload service fails too. How should the architecture be redesigned for resilience?

- A) Add more instances to the processing service
- B) Place an SQS queue between the upload and processing services; upload service sends to SQS, processing service polls from SQS
- C) Use a load balancer in front of the processing service
- D) Add retry logic in the upload service

**Answer: B**
> SQS decouples the services. The upload service places messages in SQS and immediately returns success to the user. The processing service polls SQS independently. If the processing service goes down, messages accumulate in the queue and are processed when it recovers. The upload service is unaffected — this is the core decoupling pattern.

---

### Q93 (M) — Athena
A company stores CloudTrail logs in S3 in JSON format. The security team wants to run SQL queries to analyze API activity without any additional data movement or infrastructure setup. What should be used?

- A) Load logs into RDS and run SQL queries
- B) Use Amazon Athena to query the JSON files directly in S3
- C) Use Amazon Redshift Spectrum
- D) Use AWS Glue to convert to Parquet and load into DynamoDB

**Answer: B**
> Athena is the perfect fit — serverless SQL directly on S3, no data movement required, supports JSON natively. Create a table pointing to the CloudTrail S3 prefix and query immediately. No infrastructure to manage. CloudTrail even has an Athena table template built into the console.

---

### Q94 (H) — Glue
A data engineering team receives JSON files in S3 daily from multiple sources with slightly different schemas. They need to normalize the data, convert it to Parquet format, and store it in a different S3 bucket for Athena queries. What is the BEST solution?

- A) Write a Python script on EC2 to process files nightly
- B) Use AWS Glue Crawler to discover schemas → Glue ETL job to transform and convert to Parquet → write to output S3 bucket
- C) Use Lambda to convert each file on upload
- D) Use Kinesis Data Firehose with data format conversion

**Answer: B**
> Glue Crawler automatically discovers schemas from S3 JSON files and populates the Data Catalog. A Glue ETL job can then normalize different schemas, convert to Parquet, and write to the output bucket — all serverless. Lambda has a 15-minute timeout and 10GB memory limit that may not handle large files.

---

### Q95 (M) — Disaster Recovery
A company needs a DR strategy. Their RPO is 1 hour and RTO is 4 hours. They want the lowest cost solution that meets these requirements. Which DR strategy should they choose?

- A) Multi-site active/active
- B) Warm standby
- C) Pilot light
- D) Backup and restore

**Answer: D**
> Backup and Restore: RPO determined by backup frequency (hourly backups → 1-hour RPO), RTO of several hours for restore — meets the 1hr/4hr requirement at lowest cost. Pilot Light/Warm Standby would achieve faster RTO but cost more. Multi-site is for near-zero RTO/RPO. Always match strategy to requirements at minimum cost.

---

### Q96 (H) — DR Strategies
A company currently backs up their RDS database to S3 snapshots every hour (RPO = 1 hour). They need to improve their RTO from 4 hours to under 10 minutes while keeping costs low. What should be the new DR strategy?

- A) Enable RDS Multi-AZ (changes RTO, not RPO)
- B) Move to Pilot Light — keep a minimal RDS instance in the DR region always running, using cross-region read replica
- C) Move to Warm Standby — scale up the DR environment on failure
- D) Move to Multi-Site Active/Active with Aurora Global Database

**Answer: C**
> Warm Standby keeps a scaled-down but fully functional copy of the environment running in the DR region. On failure, you scale it up (minutes, not hours). This achieves RTO of minutes. Pilot Light keeps only core services (like a small RDS), but still requires provisioning and starting other services. Multi-Site Active/Active exceeds requirements and costs more.

---

### Q97 (M) — CloudFormation
A DevOps team uses CloudFormation to manage their infrastructure. They accidentally deleted a CloudFormation stack that included an RDS database containing production data. They want to ensure the RDS instance is NOT deleted when the stack is deleted in the future. What should be configured?

- A) Enable RDS Multi-AZ to prevent deletion
- B) Set `DeletionPolicy: Retain` on the RDS resource in the CloudFormation template
- C) Add a Condition in CloudFormation to prevent deletion
- D) Enable termination protection on the CloudFormation stack

**Answer: B**
> `DeletionPolicy: Retain` on a CloudFormation resource keeps the resource (in this case RDS) even when the stack is deleted. The resource becomes unmanaged after stack deletion. Stack termination protection prevents accidental stack deletion but doesn't specify resource-level behavior. For databases, you can also use `DeletionPolicy: Snapshot` to create a final snapshot.

---

### Q98 (H) — AWS Backup
A company needs to meet compliance requirements: all EBS volumes, RDS databases, and DynamoDB tables must be backed up daily, retained for 7 years, with backups protected from deletion even by administrators. What is the BEST solution?

- A) Create individual backup scripts for each service using Lambda
- B) Use AWS Backup with a backup plan, assign resources, and enable Backup Vault Lock (Compliance mode)
- C) Take manual snapshots daily and store in S3 with Object Lock
- D) Use AWS Config to verify backups exist and alert if missing

**Answer: B**
> AWS Backup provides centralized backup management for all listed services (EBS, RDS, DynamoDB) with backup plans (schedule, retention). Backup Vault Lock in Compliance mode prevents backup deletion by anyone (including root) for the retention period — meeting the immutability requirement. This is the most operationally efficient solution.

---

### Q99 (M) — DataSync vs Storage Gateway
A company needs to migrate 50TB of data from an on-premises NFS server to Amazon EFS. After migration, the on-premises applications should continue to access the same data through the NFS protocol, but now served from EFS. What is the correct approach?

- A) Use Storage Gateway (File Gateway) for the migration, then continue using it for ongoing access
- B) Use AWS DataSync for the one-time migration to EFS, then replace the on-premises NFS server with an EFS mount target
- C) Use Snowball Edge to copy data to S3, then S3 → EFS migration
- D) Use AWS DMS to migrate NFS data

**Answer: A**
> Storage Gateway File Gateway (NFS/SMB) can serve as both the migration tool and the ongoing hybrid storage solution. On-premises applications continue to use NFS protocol without changes — requests go through the File Gateway which caches locally and stores in S3/EFS on the backend. DataSync migrates but doesn't provide ongoing NFS access from on-premises.

---

### Q100 (H) — Migration Strategy
A company wants to migrate their 3-tier web application to AWS with the LEAST risk and FASTEST timeline. The application runs on VMware VMs. They don't want to modify any code. Which migration strategy should be used?

- A) Refactor — rewrite as microservices on ECS
- B) Replatform — move to Elastic Beanstalk with minor changes
- C) Rehost (Lift and Shift) — use AWS Application Migration Service (MGN) to replicate VMs to EC2
- D) Repurchase — move to SaaS equivalent

**Answer: C**
> Rehost (Lift and Shift) using MGN replicates VMware VMs to EC2 with no code changes and minimal downtime. It's the fastest, lowest-risk approach. Refactor (rewrite) is highest effort. Replatform requires some modification. Repurchase only works if a SaaS equivalent exists.

---

### Q101 (M) — AI Services — Use Case Matching
A company wants to automatically extract key-value pairs, tables, and handwritten text from thousands of scanned PDF forms (tax returns, insurance claims) and store the structured data in a database. Which service should be used?

- A) Amazon Rekognition
- B) Amazon Comprehend
- C) Amazon Textract
- D) Amazon Transcribe

**Answer: C**
> Textract is designed specifically for extracting structured data from documents — it understands form structure (key-value pairs), tables, checkboxes, and handwriting. Rekognition is for visual content (faces, objects), not document structure. Comprehend analyzes free text for sentiment/entities, not form extraction. Transcribe is speech-to-text.

---

### Q102 (E) — AI Services — Chatbot
A company wants to add a customer service chatbot to their website that can understand customer intents like "track order," "cancel subscription," and "speak to agent." The chatbot should integrate with their backend Lambda functions. Which service should be used?

- A) Amazon Polly
- B) Amazon Lex
- C) Amazon Comprehend
- D) Amazon Rekognition

**Answer: B**
> Amazon Lex is the conversational AI service for building chatbots. It understands intents (what the user wants) and slots (required information), integrates natively with Lambda for backend fulfillment. Polly converts text to speech (output voice). Comprehend analyzes existing text. Lex is what powers Alexa.

---

### Q103 (M) — SageMaker
A data science team wants to train a custom ML model on large datasets stored in S3, then deploy it as a real-time prediction API endpoint. They want a fully managed service. What should they use?

- A) EC2 with custom ML framework and ELB in front
- B) Amazon SageMaker Training + SageMaker Endpoints
- C) AWS Glue ML Transforms
- D) Amazon Rekognition Custom Labels

**Answer: B**
> SageMaker provides end-to-end managed ML: Training Jobs pull data from S3, train on managed compute, save model artifacts to S3. SageMaker Endpoints deploy the model as a scalable real-time API. No infrastructure management. Rekognition Custom Labels is only for image classification — not general ML.

---

### Q104 (H) — Bedrock + RAG
A company has thousands of internal documents (policies, procedures, knowledge base articles) in S3. They want to build an internal assistant that answers employee questions based on this knowledge base using a large language model. Which architecture is BEST?

- A) Fine-tune a model in SageMaker on all documents
- B) Use Amazon Bedrock Knowledge Bases (RAG) pointing to the S3 documents + a foundation model
- C) Use Amazon Kendra for document indexing + custom LLM API
- D) Use Amazon Comprehend to index documents + Lex for Q&A

**Answer: B**
> Bedrock Knowledge Bases implements RAG (Retrieval-Augmented Generation) — it indexes your S3 documents into a vector store, and when a question is asked, it retrieves relevant document chunks and passes them to the FM (Claude, Titan, etc.) as context. This is more accurate than fine-tuning and keeps data current. Kendra is enterprise search (keywords), not conversational GenAI.

---

### Q105 (M) — QuickSight
A business team wants to create interactive dashboards from data in Redshift, RDS, and Athena. The dashboards should refresh automatically and be shared across 200 users. The solution should require no server management. What should be used?

- A) Grafana on EC2
- B) Amazon QuickSight
- C) Kibana on OpenSearch
- D) Tableau on EC2

**Answer: B**
> QuickSight is AWS's fully managed, serverless BI service. It natively connects to Redshift, RDS, Athena, S3, and more. Dashboards auto-refresh, scale to users without infrastructure management, and use SPICE in-memory engine. Grafana/Kibana/Tableau require server management.

---

### Q106 (H) — Kinesis Data Streams Scaling
A Kinesis Data Stream has 4 shards. The application is experiencing `ProvisionedThroughputExceededException` on one shard due to hot partition (one partition key is disproportionately popular). What are the correct solutions? (Choose TWO)

- A) Increase the number of shards (shard splitting)
- B) Use a more random/distributed partition key
- C) Enable Enhanced Fan-Out
- D) Switch to Kinesis Firehose
- E) Increase the retention period

**Answer: A and B**
> Hot partition → two fixes: (1) Split the hot shard to distribute load across more shards. (2) Use a more distributed partition key to spread records evenly. Enhanced Fan-Out increases read throughput, not write throughput. Firehose is for delivery, not stream ingestion.

---

### Q107 (M) — SQS FIFO
An e-commerce platform processes order status updates (created → payment_pending → paid → shipped → delivered). The updates must be processed in exact order per order ID. Expected volume is 500 updates/second. Which queue type should be used?

- A) SQS Standard with application-level sequencing
- B) SQS FIFO with MessageGroupId = order_id
- C) Kinesis Data Streams with partition key = order_id
- D) SNS FIFO

**Answer: B**
> SQS FIFO with `MessageGroupId = order_id` ensures messages for each order are processed in FIFO order. 500 updates/second is within FIFO limits (300 TPS native, 3,000 with batching). Kinesis would also work but FIFO SQS is simpler for this use case. Standard SQS doesn't guarantee ordering.

---

### Q108 (H) — EventBridge + Cross-Account
A multi-account company wants to centralize all security-related events (GuardDuty findings, Config compliance changes, CloudTrail events) from 20 accounts into one central security account for processing. What is the MOST efficient architecture?

- A) In each account, send findings to SNS, then each SNS fans out to the central account's SQS
- B) Enable EventBridge cross-account event bus access — each member account sends events to the central account's custom event bus; create rules in the central account
- C) Configure CloudTrail Organization Trail and export to S3 in the central account
- D) Use Security Hub with Organizations integration

**Answer: B**
> EventBridge cross-account event routing is the cleanest solution for routing events from many accounts to one. Each member account's EventBridge sends matching events to the central account's event bus. The central account creates rules to process all incoming events. This is event-driven, real-time, and scalable. Security Hub aggregates findings but doesn't replace event routing.

---

### Q109 (M) — Lambda + SQS Architecture
A company processes orders via a Lambda function triggered by SQS. Each order takes 90 seconds to process. The SQS visibility timeout is 30 seconds. What problem will occur?

- A) Lambda will process duplicate orders
- B) Messages will be deleted before Lambda finishes processing
- C) Lambda will fail to receive messages from SQS
- D) The SQS queue will stop accepting new messages

**Answer: A**
> If the visibility timeout (30s) is less than the Lambda processing time (90s), the message becomes visible again before Lambda finishes. SQS delivers the message to another Lambda invocation — resulting in duplicate processing. Fix: set visibility timeout to at least 6x the Lambda function timeout (minimum visibility = processing time + buffer).

---

### Q110 (H) — Architecture Design
A company runs a photo sharing app. Users upload photos, which are stored in S3. Each upload should trigger: (1) thumbnail generation, (2) metadata extraction, (3) content moderation. All three must happen independently and in parallel. What is the BEST architecture?

- A) S3 → Lambda with three sequential function calls
- B) S3 → SNS Topic → three SQS queues (one per process) → three Lambda functions
- C) S3 → Lambda → three parallel async Lambda invocations
- D) S3 → Step Functions with three parallel branches

**Answer: B**
> SNS fan-out to three SQS queues ensures: (1) Parallel processing — all three processes start simultaneously, (2) Decoupling — each process runs independently with its own queue, retry, and DLQ, (3) If one process fails, others are unaffected. Direct Lambda chaining (A, C) creates tight coupling. Step Functions would work but is more complex than needed.

---

### Q111 (M) — OpenSearch
A company needs to analyze application logs for error patterns, view real-time dashboards, and search through log messages. The logs are currently sent to CloudWatch Logs. What is the BEST architecture?

- A) Export CloudWatch Logs to S3 and use Athena for queries
- B) Configure a CloudWatch Logs subscription filter to stream to OpenSearch via Kinesis Firehose; use OpenSearch Dashboards for visualization
- C) Use CloudWatch Logs Insights for all log analysis
- D) Store logs in RDS and use SQL for log analysis

**Answer: B**
> OpenSearch + Dashboards is purpose-built for log analytics with full-text search, real-time dashboards, and complex log pattern analysis. Kinesis Firehose streams from CloudWatch Logs to OpenSearch efficiently. CloudWatch Logs Insights is fine for occasional queries but not for persistent dashboards or deep full-text search.

---

### Q112 (E) — Snow Family
A company needs to transfer 10PB of data to AWS. Their internet connection is 1 Gbps. Approximately how long would a network transfer take, and which Snow device is appropriate?

- A) ~22 hours; use Snowball Edge
- B) ~100 days; use multiple Snowball Edge devices
- C) ~22 days; use Snowmobile
- D) ~1,000 days; use Snowmobile

**Answer: D — use Snowmobile**
> 10PB / (1Gbps = 125MB/s) = 80,000,000 seconds ≈ 926 days ≈ ~2.5 years. Network transfer is infeasible. A single Snowball Edge holds 80TB — you'd need 125+ devices. Snowmobile holds 100PB (one truck) and is designed exactly for exabyte/petabyte-scale migrations at this level.

---

### Q113 (H) — Well-Architected: Cost
A company runs a web application on 20 on-demand EC2 instances 24/7. Analysis shows that baseline traffic requires 10 instances, and the other 10 are only needed during 8-hour business hours. What purchasing optimization should be recommended?

- A) Convert all 20 to Reserved Instances
- B) Convert the baseline 10 to 1-year Reserved Instances (or Compute Savings Plans); use Spot Instances for the additional demand
- C) Convert all 20 to Spot Instances
- D) Use Scheduled Reserved Instances for the extra 10

**Answer: B**
> Best practice for cost optimization: Reserve the known steady-state (10 always-on) with Reserved Instances or Savings Plans (up to 72% savings). The additional demand (8 hours/day) can use Spot (if fault-tolerant) or On-Demand. Converting all to Reserved wastes money on capacity not needed 16 hours/day. All Spot risks interruptions for the baseline.

---

### Q114 (M) — Rekognition
A media company wants to automatically detect and blur faces in user-uploaded videos before publishing them. Which service handles this?

- A) Amazon Comprehend
- B) Amazon Textract
- C) Amazon Rekognition Video
- D) Amazon Transcribe

**Answer: C**
> Rekognition Video can detect and locate faces in video streams with bounding boxes and timestamps. The company can use this data to programmatically blur the detected face regions in each frame. Comprehend handles text, Textract handles documents, Transcribe handles audio.

---

### Q115 (H) — Multi-Region Architecture
A company wants to build a globally available web application with automatic failover. The primary region is us-east-1. Users should be automatically routed to eu-west-1 if us-east-1 is unhealthy. The solution must have the lowest possible latency for users worldwide. What architecture should be used?

- A) Route 53 Failover → ALB in us-east-1 only
- B) Route 53 Latency routing → ALB (us-east-1) + ALB (eu-west-1) with health checks enabled on both records
- C) CloudFront → multi-origin with origin failover (us-east-1 primary, eu-west-1 secondary)
- D) Global Accelerator → endpoints in both regions with health checks

**Answer: B**
> Route 53 Latency routing directs users to the region with lowest latency AND with health checks enabled, it performs failover if a region is unhealthy. Users in the Americas go to us-east-1, users in Europe go to eu-west-1, and if us-east-1 fails, all users fail over to eu-west-1. This provides both performance and availability.

---

### Q116 (M) — Serverless Architecture
A startup wants to build a REST API with the following requirements: no server management, auto-scaling to handle any traffic volume, pay only for actual requests, and integrate with a NoSQL database. What is the MOST appropriate stack?

- A) EC2 + Auto Scaling + RDS
- B) Elastic Beanstalk + RDS PostgreSQL
- C) API Gateway + Lambda + DynamoDB
- D) ECS Fargate + Aurora Serverless

**Answer: C**
> API Gateway + Lambda + DynamoDB is the canonical serverless stack: fully managed, auto-scales to any volume, pay-per-request pricing, no servers to manage. ECS Fargate/Aurora Serverless v2 are managed but still have baseline costs and more operational complexity.

---

### Q117 (H) — Final Boss: Architecture Review
A company processes customer transactions. The architecture must: (1) accept orders via REST API, (2) process payments asynchronously, (3) send confirmation emails, (4) store all transactions in a database queryable via SQL, (5) archive transactions older than 1 year to cold storage, (6) handle 10,000 transactions/second at peak. Which architecture BEST meets all requirements?

- A) EC2 fleet → SQS → EC2 payment processor → RDS → S3 Glacier lifecycle
- B) API Gateway → Lambda → SQS → Lambda (payment) → SNS (email) → Aurora → S3 Glacier (lifecycle policy on Aurora backups)
- C) API Gateway → Lambda → SQS FIFO → Lambda (payment) → SES (email) → Aurora → S3 (lifecycle: Standard → Glacier Deep Archive after 365 days using Aurora export to S3)
- D) API Gateway → EC2 → Kinesis → EC2 fleet → RDS → Glacier Vault

**Answer: C**
> Breaking it down: (1) API Gateway REST API → Lambda, (2) SQS FIFO ensures payment ordering, Lambda processes async, (3) SES (Simple Email Service) for transactional emails, (4) Aurora is serverless SQL scaling to 10K TPS, (5) Export Aurora to S3 + lifecycle policy transitions to Glacier Deep Archive after 1 year. Option B uses SNS for email — SES is the correct transactional email service. Option A/D have EC2 infrastructure overhead.

---

## Answer Key Summary

| Q# | Answer | Q# | Answer | Q# | Answer |
|----|--------|----|--------|----|--------|
| Q85 | B | Q96 | C | Q107 | B |
| Q86 | C | Q97 | B | Q108 | B |
| Q87 | C | Q98 | B | Q109 | A |
| Q88 | C | Q99 | A | Q110 | B |
| Q89 | B | Q100 | C | Q111 | B |
| Q90 | B | Q101 | C | Q112 | D |
| Q91 | C | Q102 | B | Q113 | B |
| Q92 | B | Q103 | B | Q114 | C |
| Q93 | B | Q104 | B | Q115 | B |
| Q94 | B | Q105 | B | Q116 | C |
| Q95 | D | Q106 | A+B | Q117 | C |

---

**Score: __ / 33**
