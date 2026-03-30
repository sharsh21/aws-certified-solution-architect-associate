# Integration & Messaging Services — SAA-C03 Exam Guide

---

## SQS — Simple Queue Service

### Core Concepts
- Fully managed **message queue** (producer → queue → consumer)
- Pull-based (consumers poll)
- **Message retention:** 4 days default, up to **14 days**
- **Message size:** Max **256KB** (larger via SQS Extended Client with S3)
- **Visibility Timeout:** Time message is hidden after being received (default 30s, max 12hr)
- **In-flight messages:** Being processed but not yet deleted
- At-least-once delivery (can receive duplicate messages)

### Queue Types

| | Standard Queue | FIFO Queue |
|-|---------------|-----------|
| **Order** | Best-effort (not guaranteed) | Strictly FIFO |
| **Delivery** | At-least-once (duplicates possible) | Exactly-once processing |
| **Throughput** | Nearly unlimited | 300 msg/s (3,000 with batching) |
| **Use Case** | Decoupling, high throughput | Ordering critical (banking, inventory) |
| **Naming** | Any name | Must end in `.fifo` |

### SQS Key Features
- **Dead Letter Queue (DLQ):** Failed messages after N receive attempts go to DLQ
- **Delay Queue:** Delay delivery up to 15 minutes (Message Timer overrides per message)
- **Long Polling:** Wait up to 20 seconds for messages — reduces empty responses, costs
- **Short Polling:** Return immediately (even if empty) — more API calls, less efficient
- **Message Groups (FIFO):** Parallel FIFO within same queue (MessageGroupId)
- **Deduplication (FIFO):** ContentBased or MessageDeduplicationId (5-minute window)

### SQS Extended Client Library
- Stores message payload in S3
- Sends reference to S3 object in SQS
- For messages > 256KB

### SQS as Lambda Trigger
- Lambda polls SQS in batches (event source mapping)
- Batch size: 1-10,000 (Standard), 1-10 (FIFO)
- Failed batch → goes to DLQ (configure on SQS, not Lambda)
- **Lambda Scaling:** One Lambda invocation per batch, scales to handle queue depth

### SQS Security
- SSE with KMS
- VPC Endpoint (Interface) for private access
- Resource policy for cross-account access

---

## SNS — Simple Notification Service

### Core Concepts
- **Pub/Sub** (publish once → deliver to many subscribers)
- Push-based
- **Topic:** Named channel for messages
- **Subscriptions:** Endpoints subscribed to a topic
- Messages delivered immediately to all subscribers

### SNS Subscribers
- **Protocols:** HTTP/HTTPS, Email, Email-JSON, SMS, SQS, Lambda, Mobile Push (APNs, GCM), Kinesis Firehose

### SNS Message Filtering
- Filter policy on subscription (JSON)
- Only deliver messages matching filter (reduce costs, processing)

### SNS Message Fan-Out Pattern
```
App → SNS Topic → SQS Queue 1 (email service)
                → SQS Queue 2 (fraud service)
                → Lambda (real-time processing)
```
> **Exam Tip:** Fan-out = SNS + multiple SQS queues. This enables parallel processing with decoupling.

### SNS FIFO
- Strict ordering + deduplication
- Only SQS FIFO as subscriber
- 300 msg/s throughput

### SNS Security
- SSE with KMS
- Resource policy for cross-account
- VPC Interface Endpoint

---

## SQS vs SNS vs EventBridge

| | SQS | SNS | EventBridge |
|-|-----|-----|-------------|
| **Model** | Queue (pull) | Pub/Sub (push) | Event bus (push) |
| **Consumers** | One consumer per message | All subscribers | Rules-based routing |
| **Persistence** | Up to 14 days | None (immediate) | None (immediate) |
| **Filtering** | No | Message filter policy | Rich content filtering |
| **Sources** | App-generated | App-generated | AWS services + custom + SaaS |
| **Use Case** | Async decoupling | Fan-out notifications | Event-driven, AWS service events |

---

## EventBridge (formerly CloudWatch Events)

### Core Concepts
- Serverless event bus
- Connect AWS services, custom apps, SaaS (Zendesk, Datadog, etc.)
- **Event Buses:**
  - **Default:** AWS service events
  - **Custom:** Your app events
  - **Partner:** SaaS provider events

### EventBridge Components
- **Event:** JSON payload describing what happened
- **Rule:** Pattern match → route to target
- **Target:** Lambda, SQS, SNS, Kinesis, Step Functions, EC2, ECS, API Gateway, etc.

### Scheduling
- Cron expressions or rate expressions
- Schedule Lambda, SSM Automation, etc.

### EventBridge Pipes
- Point-to-point integration with filtering, enrichment, transformation
- Source → Filter → Enrich → Target

### EventBridge Schema Registry
- Discover and track event schemas
- Generate code bindings (Java, Python, TypeScript)

---

## Kinesis

### Kinesis Family
| Service | Purpose |
|---------|---------|
| **Kinesis Data Streams** | Collect and process real-time streaming data |
| **Kinesis Data Firehose** | Load streaming data to S3, Redshift, OpenSearch, Splunk |
| **Kinesis Data Analytics** | SQL or Apache Flink analysis on streaming data |
| **Kinesis Video Streams** | Capture and process video streams |

### Kinesis Data Streams

- **Shards:** Capacity unit — 1MB/s ingestion, 2MB/s consumption per shard
- **Data retention:** 24 hours default, up to 365 days
- **Ordering:** Guaranteed within a shard (use partition key to route related records to same shard)
- **Consumers:** Standard (pull, 2MB/s/shard shared) or Enhanced Fan-Out (push, 2MB/s/shard per consumer)
- **Sequence Number:** Unique per record per shard
- Supports replay (unlike SQS)

### Kinesis Data Firehose
- **Fully managed**, no shards to manage
- **Near real-time** (60-second buffer or 1MB buffer)
- Auto-scaling
- **Destinations:** S3, Redshift (via S3), OpenSearch, Splunk, HTTP endpoints, Datadog
- Can transform data with Lambda
- NOT for real-time — use KDS for real-time

### Kinesis vs SQS

| | Kinesis Data Streams | SQS |
|-|---------------------|-----|
| **Ordering** | Per shard | FIFO queue only |
| **Replay** | Yes (up to 365 days) | No (once consumed and deleted) |
| **Consumers** | Multiple (fan-out) | One consumer per message |
| **Routing** | Partition key → shard | N/A |
| **Use Case** | Real-time analytics, log processing | Async decoupling, task queue |

---

## Step Functions

- **Serverless orchestration** for workflows (state machines)
- Visual workflow using JSON/YAML (Amazon States Language)
- **States:** Task, Choice, Wait, Parallel, Map, Pass, Succeed, Fail

### Workflow Types
| | Standard | Express |
|-|---------|---------|
| Max duration | 1 year | 5 minutes |
| Execution model | Exactly-once | At-least-once |
| Execution history | Audit trail | CloudWatch only |
| Pricing | Per state transition | Per execution duration |
| Use Case | Long-running, business processes | High-volume, IoT, streaming |

### Step Functions Integrations
- Lambda, DynamoDB, S3, SQS, SNS, ECS, Batch, SageMaker, Glue, Athena
- **Optimistic integrations (SDK):** Call and continue
- **Optimistic integrations (waitForTaskToken):** Wait for external callback
- **Run a job (.sync):** Wait for job to complete (Glue, ECS, Batch)

---

## AppSync

- Managed **GraphQL API** service
- Real-time data with WebSocket subscriptions
- Offline sync for mobile apps
- Sources: DynamoDB, Lambda, RDS, HTTP APIs, OpenSearch
- Authentication: IAM, Cognito, OIDC, API Key

---

## Amazon MQ

- Managed **ActiveMQ and RabbitMQ**
- Use when **migrating existing on-premises message broker** (MQTT, AMQP, STOMP, OpenWire, WSS)
- **Not cloud-native** — if building new, use SQS/SNS
- Multi-AZ with failover

> **Exam Tip:** If migrating existing JMS or AMQP application → Amazon MQ. Building new cloud-native messaging → SQS/SNS.

---

## Architecture Patterns

### Decoupling Pattern
```
Producer (EC2/Lambda) → SQS Queue → Consumer (EC2/Lambda)
```
- If producer is faster than consumer: queue builds up, consumer auto-scales

### Fan-Out Pattern
```
Event → SNS Topic → SQS Queue A (app team A)
                 → SQS Queue B (app team B)
                 → Lambda (notifications)
```

### Event-Driven Architecture
```
User Action → EventBridge → Rule 1 → Lambda (process)
                          → Rule 2 → SQS (queue for retry)
                          → Rule 3 → Step Functions (workflow)
```

### Real-Time Processing Pipeline
```
Data → Kinesis Data Streams → Lambda (real-time)
                           → Kinesis Firehose → S3 (batch analytics)
```

---

## Key Exam Traps

1. **SQS message deleted only when consumer deletes it** — visibility timeout hides, doesn't delete
2. **SNS push, SQS pull** — SNS cannot save messages; if no subscriber, message is lost
3. **Kinesis Data Firehose is near real-time** (60s minimum buffer), NOT real-time
4. **FIFO SQS max 300 TPS** (3,000 with batching) — not suitable for very high throughput
5. **DLQ must be same type** as source queue (FIFO DLQ for FIFO queue)
6. **Amazon MQ = existing app migration**, not for new cloud apps
7. **SQS doesn't guarantee ordering** in standard queue — use FIFO if ordering matters
8. **EventBridge = rich filtering from AWS services**, SNS = simple fan-out
