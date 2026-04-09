# AI & ML Services — SAA-C03 Exam Guide

> **Exam depth:** High-level only. Know WHAT each service does and WHEN to use it.
> Typically 2-4 questions. Focus on use-case matching, not ML internals.

---

## Core AI/ML Services — Use Case Mapping (MEMORIZE)

| Service | What It Does | Key Use Case |
|---------|-------------|-------------|
| **SageMaker** | Build, train, deploy ML models | Custom ML models, end-to-end ML platform |
| **Rekognition** | Image & video analysis (CV) | Face detection, object labeling, content moderation |
| **Comprehend** | NLP — text analysis | Sentiment analysis, entity extraction, topic modeling |
| **Translate** | Real-time language translation | Multilingual apps, content localization |
| **Transcribe** | Speech → text (ASR) | Call transcription, subtitles, voice search |
| **Polly** | Text → speech (TTS) | Voice-enabled apps, accessibility, podcasts |
| **Lex** | Conversational AI (chatbot) | Chatbots, voice bots (powers Alexa) |
| **Kendra** | Intelligent enterprise search | Search across documents, SharePoint, S3, databases |
| **Personalize** | Real-time personalized recommendations | E-commerce recommendations, content personalization |
| **Forecast** | Time-series forecasting | Demand planning, inventory, financial forecasting |
| **Textract** | Extract text & data from documents | Forms, invoices, ID cards (beyond basic OCR) |
| **Bedrock** | Foundation models / Generative AI API | GenAI apps, LLM integration, no ML expertise needed |
| **CodeWhisperer** | AI code suggestions in IDE | Developer productivity |

---

## SageMaker — Deeper Detail (Most Likely Tested)

### Key Components
- **SageMaker Studio:** Unified ML IDE
- **SageMaker Ground Truth:** Data labeling with human/automated workflows
- **SageMaker Training Jobs:** Managed ML training (EC2 under the hood)
- **SageMaker Endpoints:** Deploy models for real-time inference
- **SageMaker Batch Transform:** Offline/batch inference on large datasets
- **SageMaker Pipelines:** MLOps CI/CD for ML
- **SageMaker Feature Store:** Centralized feature repository
- **SageMaker Model Monitor:** Detect data/model drift in production

### Storage for SageMaker
- Training data: **S3** (most common)
- Feature store: **SageMaker Feature Store**
- Models stored in **S3**
- **EFS/FSx for Lustre** for high-performance training (low-latency data access)

### SageMaker + Architecture
```
S3 (training data) → SageMaker Training → S3 (model artifacts) → SageMaker Endpoint → API Gateway → App
```

### Real-World Use Cases & When to Use SageMaker

> **The problem it solves:** End-to-end ML platform — data labeling → feature engineering → model training → deployment → monitoring — without managing ML infrastructure.

**Real-World Scenarios:**
| Business Problem | SageMaker Solution |
|-----------------|-------------------|
| Bank builds fraud detection model — data scientists need ML infrastructure | SageMaker Studio (IDE) + Training Jobs (managed EC2) + Endpoints (real-time inference API) |
| Retail company needs demand forecasting for 100K SKUs | SageMaker with Amazon's built-in DeepAR algorithm — trains on historical sales, deploys forecast endpoint |
| 1M product images need to be manually labeled for training | **SageMaker Ground Truth** — human labelers + automatic labeling for high-confidence items |
| MLOps: retrain model weekly with new data automatically | **SageMaker Pipelines** — CI/CD for ML: data prep → train → evaluate → conditional deploy |
| Model accuracy drops after 3 months (data drift) | **SageMaker Model Monitor** — detects input data drift, statistical deviation from training distribution |
| High-throughput batch inference on 10M records overnight | **SageMaker Batch Transform** — no endpoint needed, run inference on entire dataset, write to S3 |

---

## Rekognition — Key Features
- **Image:** Object detection, facial analysis, face comparison, celebrity recognition, text in images
- **Video:** Activity detection, person tracking, content moderation (unsafe content)
- **Face Liveness:** Detect real user vs photo/video (for auth)
- Serverless — no ML knowledge needed
- Input: S3 images/videos or real-time streams (Kinesis Video Streams)

### Real-World Use Cases
| Business Problem | Rekognition Solution |
|-----------------|---------------------|
| Social media platform: user uploads photos — detect nudity automatically | Rekognition `DetectModerationLabels` → flag if `Explicit Nudity` confidence > 90% → reject upload |
| Smart building: identify employees at door (face recognition) | Rekognition `SearchFacesByImage` against employee face collection — grant/deny access |
| Insurance company processes accident photos — count damaged cars | Rekognition `DetectLabels` → count "Car" labels → route to appropriate claim handler |
| Streaming app: verify user is a real person (not a photo) for age verification | Rekognition **Face Liveness** — detects spoof attempts (photo/video replay attacks) |
| Sports highlight reel: find all clips featuring a specific athlete | Rekognition Video `SearchFaces` on recorded match — timestamps of all appearances |

---

## Comprehend — Key Features
- Detects: language, entities (people, places, dates), key phrases, sentiment, PII
- **Comprehend Medical:** Extract medical info from unstructured text (ICD-10 codes, medications)
- Custom classifiers and entity recognizers (train on your data)

### Real-World Use Cases
| Business Problem | Comprehend Solution |
|-----------------|---------------------|
| Customer support: 10,000 reviews/day — automatically categorize by sentiment | Comprehend `DetectSentiment` → positive/negative/neutral/mixed → route to teams |
| Legal firm: search contracts for entities (company names, dates, dollar amounts) | Comprehend `DetectEntities` — extracts structured info from unstructured legal text |
| Healthcare: clinical notes have medication names and dosages in free text | **Comprehend Medical** — identifies medications, dosages, ICD-10 codes, named medical entities |
| GDPR compliance: scan millions of S3 documents for PII | Comprehend PII detection → flag documents containing email, SSN, phone numbers |
| Support tickets arrive in 30 languages — route to correct regional team | Comprehend `DetectDominantLanguage` → identify language → route to correct agent queue |

---

## Textract vs Rekognition vs Comprehend

| | Rekognition | Textract | Comprehend |
|-|------------|---------|-----------|
| Input | Images/video | Documents/forms/PDFs | Text |
| Output | Labels, faces, objects | Structured text, tables, forms | Entities, sentiment, key phrases |
| Use Case | Visual analysis | Document digitization | Text analytics/NLP |

> **Exam Tip:** If extracting data from a scanned invoice or form → **Textract**. If analyzing an image for objects/faces → **Rekognition**. If analyzing text for sentiment/entities → **Comprehend**.

---

## Amazon Lex — Key Facts
- Same tech as Alexa
- **Intents** (what user wants) + **Slots** (parameters)
- Integrates with Lambda for fulfillment
- Can be embedded in web, mobile, contact centers (Amazon Connect)

## Amazon Connect
- Cloud-based **contact center** service
- Integrates with Lex (IVR/chatbot), Lambda, DynamoDB, S3
- Pay per minute — no upfront costs

---

## Amazon Bedrock — Key Facts (Newer, Increasingly Tested)
- Access foundation models (FMs) via API — no ML infrastructure
- Models from: Anthropic (Claude), Meta (Llama), Stability AI, Amazon (Titan), Mistral, Cohere
- **RAG (Retrieval-Augmented Generation):** Bedrock Knowledge Bases — connect FMs to your data (S3)
- **Bedrock Agents:** Orchestrate multi-step tasks using FMs + tools
- **Model Evaluation:** Compare model outputs
- Fully serverless — pay per token

### Real-World Use Cases & When to Use Bedrock

> **The problem it solves:** Build GenAI applications using world-class foundation models via a simple API — no GPU clusters, no model training, no ML expertise needed.

**Choose Bedrock when:** You want GenAI capabilities (summarization, generation, Q&A, code generation) using pre-trained models. For custom ML on your own data → SageMaker.

**Real-World Scenarios:**
| Business Problem | Bedrock Solution |
|-----------------|----------------|
| Internal chatbot that answers questions from company policy documents (PDF/S3) | **Bedrock Knowledge Bases** (RAG) — embeddings stored in vector DB, Claude answers from docs |
| Customer service bot that automatically handles returns, tracks orders, updates records | **Bedrock Agents** — multi-step: calls Lambda (check order), DynamoDB (update status), sends email |
| Company needs to evaluate Claude vs Titan vs Llama for their use case | Bedrock **Model Evaluation** — run same prompts through all models, compare output quality |
| Legal team needs 100-page contracts summarized — no ML team | Bedrock API (Claude) — `POST /model/invoke` with contract text → summary in seconds |
| E-commerce: generate 10,000 unique product descriptions from specs | Batch inference with Bedrock Titan Text — structured input → creative descriptions at scale |

**Bedrock vs SageMaker for the exam:** Bedrock = use existing Foundation Models (no training, no ML expertise). SageMaker = build/train/deploy custom ML models (your data, your architecture).

---

## Quick Decision Tree for AI Services

```
Need custom ML model?           → SageMaker
Need GenAI / LLM?               → Bedrock
Analyze images/video?           → Rekognition
Extract text from documents?    → Textract
Analyze text (sentiment, NLP)?  → Comprehend
Speech to text?                 → Transcribe
Text to speech?                 → Polly
Build a chatbot?                → Lex
Language translation?           → Translate
Search across enterprise docs?  → Kendra
Personalized recommendations?  → Personalize
Time-series forecasting?        → Forecast
```

### Real-World Scenario Mapping for Remaining Services

| Service | Real-World Problem It Solves |
|---------|------------------------------|
| **Transcribe** | Call center records 1M calls/month — compliance requires all calls transcribed and searchable. Transcribe → S3 → Athena for search. Speaker diarization identifies agent vs customer. |
| **Polly** | Banking app needs to read transaction alerts aloud to visually impaired users. Polly TTS → MP3 audio → played in mobile app. Neural TTS voices sound natural. |
| **Translate** | Global e-commerce platform with product descriptions in English — auto-translate to 75 languages for international expansion. Translate API + Lambda → 75 language versions in S3. |
| **Lex** | Insurance company wants 24/7 claims chatbot — replaces call center for 80% of routine queries. Lex (understands intent) + Lambda (processes claim) + DynamoDB (stores claim). |
| **Kendra** | Enterprise has 500TB of documents across SharePoint, S3, Confluence — employees can't find anything. Kendra indexes all sources → natural language search: "what is the vacation policy?" |
| **Personalize** | Streaming video platform wants Netflix-style "Because you watched X" recommendations. Personalize trains on viewing history → real-time recommendations via API. |
| **Forecast** | Retailer needs to predict demand for 50,000 SKUs 3 months ahead for inventory planning. Forecast trains on 3 years of sales data → probabilistic forecasts per SKU. |
| **Textract** | Hospital digitizes 1M paper patient intake forms — needs structured data extracted. Textract extracts form field key-value pairs automatically (better than OCR). |
| **Connect** | Company replaces on-premises call center with cloud solution. Connect provides phone numbers, IVR (Lex), agent routing, recordings to S3, pay-per-minute. |

---

## Architecture Patterns

### Document Processing Pipeline
```
S3 (uploaded docs) → Lambda → Textract (extract) → Comprehend (analyze) → DynamoDB (store) → QuickSight (dashboard)
```

### Content Moderation
```
S3 (user uploads) → Lambda → Rekognition (moderation labels) → SNS (alert if flagged) → Human review
```

### GenAI RAG Application
```
S3 (knowledge base) → Bedrock Knowledge Base → Bedrock Agent → Claude/Titan FM → API Gateway → Frontend
```

### Real-Time Transcription + Analysis
```
Audio → Kinesis Video Streams / S3 → Transcribe → Comprehend → DynamoDB
```

---

## Key Exam Traps

1. **Lex ≠ Polly** — Lex = understands speech (input), Polly = generates speech (output)
2. **Textract > OCR** — it understands forms, tables, key-value pairs, not just raw text
3. **Rekognition is NOT real-time video by default** — needs Kinesis Video Streams for live video
4. **SageMaker Ground Truth** — data labeling, not model training
5. **Bedrock = no-code FM access** — for custom training, use SageMaker
6. **Comprehend Medical** ≠ Comprehend — separate service optimized for healthcare data
