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

---

## Rekognition — Key Features
- **Image:** Object detection, facial analysis, face comparison, celebrity recognition, text in images
- **Video:** Activity detection, person tracking, content moderation (unsafe content)
- **Face Liveness:** Detect real user vs photo/video (for auth)
- Serverless — no ML knowledge needed
- Input: S3 images/videos or real-time streams (Kinesis Video Streams)

---

## Comprehend — Key Features
- Detects: language, entities (people, places, dates), key phrases, sentiment, PII
- **Comprehend Medical:** Extract medical info from unstructured text (ICD-10 codes, medications)
- Custom classifiers and entity recognizers (train on your data)

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
