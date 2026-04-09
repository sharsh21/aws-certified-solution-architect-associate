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

### Real-World Use Cases & When to Use SQS

> **The problem it solves:** Decouple producers from consumers — the producer sends a message and doesn't care how fast the consumer processes it. Absorbs traffic spikes, enables independent scaling, and provides resilience when consumers go down.

**Choose Standard Queue:** High-throughput, order doesn't matter (order processing with idempotent consumers, image processing).
**Choose FIFO Queue:** Order is critical AND at-most-once delivery (payment transactions, inventory updates).

**Real-World Scenarios:**
| Business Problem | SQS Solution |
|-----------------|-------------|
| E-commerce: order service is 10x faster than fulfillment service — orders pile up | SQS Standard Queue — orders buffer in queue, fulfillment consumers process at their own pace |
| Image upload service: 10,000 images arrive in 5 min — processing takes 30 min total | SQS + Lambda (event source mapping) — Lambda scales to process queue depth |
| Payment processing — each transaction must be processed exactly once in order | **FIFO Queue** — strict ordering, deduplication, exactly-once processing |
| Failed email notifications: don't lose them, retry up to 3 times | DLQ configured — after 3 `maxReceiveCount` failures, message goes to DLQ for investigation |
| Tasks are large (500MB video files) but SQS max is 256KB | **SQS Extended Client Library** — stores payload in S3, SQS message has S3 pointer |
| Consumers process slowly — don't want to poll constantly and waste API calls | **Long Polling** — `WaitTimeSeconds=20` — reduces empty responses, cuts costs |

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

### Real-World Use Cases & When to Use SNS

> **The problem it solves:** Push one message and deliver it to ALL subscribers simultaneously — notifications, fan-out to multiple services, or cross-service event broadcasting.

**Real-World Scenarios:**
| Business Problem | SNS Solution |
|-----------------|-------------|
| Order placed — need to notify email, SMS, and trigger 3 microservices | SNS Topic → subscribers: email, SMS, SQS Queue (fraud service), SQS Queue (inventory), Lambda |
| Ops team needs critical alert for RDS failover sent to Slack + PagerDuty | CloudWatch Alarm → SNS → HTTP/HTTPS endpoint (Slack webhook) + Lambda (PagerDuty API call) |
| Multi-account architecture: security event in account A must alert account B | SNS with resource policy for cross-account subscription |
| Message filtering: fraud service only needs orders > $1000 | SNS **Message Filter Policy** on fraud SQS subscription — `{"amount": [{"numeric": [">", 1000]}]}` |
| Mobile app: push notification to 1M iOS/Android devices | SNS Mobile Push (APNs + GCM/FCM) — single API call fans out to all devices |

**The fan-out pattern (most exam-tested SNS pattern):**
Single event → SNS Topic → multiple SQS Queues → independent consumer teams process in parallel. This enables decoupling + parallel processing without tight coupling.

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

### Real-World Use Cases & When to Use EventBridge

> **The problem it solves:** React to events from AWS services, your own apps, and SaaS tools — route events to the right targets based on content without polling or tight coupling.

**Choose EventBridge when:** You need rich content-based filtering, AWS service event integration (EC2 state changes, S3 events), or SaaS integration.

**Real-World Scenarios:**
| Business Problem | EventBridge Solution |
|-----------------|---------------------|
| EC2 instance stops unexpectedly — auto-notify team and restart | EventBridge rule: `EC2 Instance State-change → state=stopped` → Lambda (restart + SNS alert) |
| Cron: generate daily reports at 6 AM without a cron server | EventBridge Schedule: `cron(0 6 * * ? *)` → Lambda → generates and emails report |
| Salesforce opportunity closes — trigger AWS order fulfillment workflow | EventBridge **Partner Event Bus** (Salesforce) → rule → Step Functions workflow |
| Microservice A emits `OrderPlaced` event — route to fulfillment, fraud, analytics | **Custom Event Bus** → 3 rules → 3 different targets (Lambda, SQS, Kinesis) |
| S3 objects created — need sophisticated routing (only `.jpg` files from prod bucket) | EventBridge S3 event notification → content filter → only routes matching pattern |

**EventBridge vs SNS for the exam:** EventBridge = richer filtering, AWS service events, SaaS integration, cron scheduling. SNS = simple fan-out/pub-sub, mobile push, direct subscriber delivery.

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

### Real-World Use Cases & When to Use Kinesis

> **The problem it solves:** Ingest and process massive streams of real-time data (logs, IoT telemetry, clickstreams, transactions) — multiple consumers can process the same stream simultaneously, and data can be replayed.

**Kinesis Data Streams (KDS) vs Kinesis Firehose:**
- KDS = you write consumer code, real-time (< 1 second), replayable
- Firehose = fully managed delivery to S3/Redshift/OpenSearch, near real-time (60s buffer), no consumer code

**Real-World Scenarios:**
| Business Problem | Kinesis Solution |
|-----------------|----------------|
| Mobile app clickstream: 1M events/sec — need real-time fraud detection + batch reporting | KDS → Consumer 1 (Lambda, real-time fraud) + Consumer 2 (Firehose → S3, batch analytics) |
| Ad tech: 500K ad impressions/sec must be analyzed in < 1 second | KDS + Lambda (Enhanced Fan-Out, 2MB/s per consumer) — real-time bid optimization |
| IoT factory: 10,000 sensors — need last 7 days of data for ML replay | KDS with 7-day retention — ML team can replay entire stream for model training |
| Log aggregation: 100 EC2 instances → need logs in OpenSearch in < 1 min | **Kinesis Firehose** — buffered delivery to OpenSearch, transform with Lambda, no code |
| Stock market feed: must process trades in the order they arrive per stock | KDS — use stock ticker as partition key → all trades for AAPL land in same shard, ordered |

**Shard capacity math for the exam:** 1 shard = 1 MB/s in + 2 MB/s out. Need 5 MB/s ingestion = 5 shards minimum.

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

### Real-World Use Cases & When to Use Step Functions

> **The problem it solves:** Orchestrate complex multi-step workflows with error handling, retries, parallel branches, and human approval steps — without building your own state machine with queues and polling.

**Choose Standard Workflow:** Long-running (days/weeks), business processes, audit trail needed.
**Choose Express Workflow:** High-volume (IoT, streaming), short duration, cost-sensitive.

**Real-World Scenarios:**
| Business Problem | Step Functions Solution |
|-----------------|------------------------|
| E-commerce order: verify payment → reserve inventory → ship → notify customer | Standard Workflow with 5 Lambda states — if payment fails, compensate (cancel reservation) |
| ML pipeline: ETL → train model → evaluate → if accuracy > 90%, deploy → else alert | Workflow with Choice state (branch on metric), Parallel state (train multiple models simultaneously) |
| Document approval: legal review needed — wait days for human approval | `waitForTaskToken` — workflow pauses, sends token via email, resumes when reviewer clicks approve |
| Nightly ETL: run Glue job → wait for it → run Athena query → store result | Step Functions `.sync` integration — waits for Glue/Athena to complete before next step |
| Video processing: transcode + thumbnail + metadata extraction (all independent) | **Parallel state** — three Lambda functions run simultaneously, workflow waits for all to finish |

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

### Real-World Use Cases & When to Use Amazon MQ

> **The problem it solves:** Managed ActiveMQ and RabbitMQ — let you migrate existing on-premises message broker workloads to AWS without rewriting your application.

**Choose Amazon MQ when:** Migrating existing JMS, AMQP, STOMP, MQTT, or OpenWire applications that you can't refactor. For NEW applications, always use SQS/SNS.

**Real-World Scenarios:**
| Business Problem | Amazon MQ Solution |
|-----------------|-------------------|
| Legacy Java EE app uses ActiveMQ with JMS — migrating to AWS | Amazon MQ (ActiveMQ) — drop-in replacement, no code changes, same JMS API |
| Banking system uses RabbitMQ for inter-service messaging — must migrate | Amazon MQ (RabbitMQ) — same AMQP protocol, exchange/queue topology preserved |
| Factory automation uses MQTT protocol for sensor data — needs HA | Amazon MQ with Multi-AZ failover — automatic failover, MQTT protocol support |

**The exam rule:** "Existing on-premises broker, AMQP/JMS/STOMP" → Amazon MQ. "Building new microservices messaging" → SQS/SNS. Amazon MQ is specifically for the migration/lift-and-shift use case.

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
