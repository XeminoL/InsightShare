---
title: "Overview"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### About InsightShare

**InsightShare** is a serverless web application for uploading, analyzing and sharing images and documents on AWS. It uses ready-to-call AWS AI services to understand each file on upload, so users search by content and share through time-limited links.

The platform is built on:
- **Amazon S3 + presigned URLs**: private file storage; the browser uploads/downloads directly through short-lived signed links.
- **AWS Lambda + Amazon API Gateway**: a Python back-end with no servers to manage, exposed as an HTTP API.
- **Amazon DynamoDB**: file metadata with AI labels and extracted text, powering content-based search.
- **Amazon Rekognition, Textract and Polly**: the AI layer, labels/moderates images, extracts document text, turns text into audio, no model training.
- **Amazon CloudFront + CloudWatch + IAM**: HTTPS delivery of the static frontend, monitoring, and least-privilege access control.

#### Workshop Overview

InsightShare is built end to end across the following parts:

- A secure file storage layer on **Amazon S3** with **presigned URLs**.
- A back-end on **AWS Lambda** (Python) behind **Amazon API Gateway**, with metadata in **Amazon DynamoDB**.
- An **AI layer** that reads file content automatically: image labels and moderation with **Amazon Rekognition**, text extraction with **Amazon Textract**, text-to-speech with **Amazon Polly**.
- **Content search** over image labels and text extracted from documents.
- The static frontend on **Amazon S3 + CloudFront** (HTTPS).
- Monitoring and security with **Amazon CloudWatch** and a least-privilege **IAM Role**.

#### Architecture Overview

InsightShare follows a fully serverless flow:

1. **Frontend delivery**: the browser loads the static web interface from **Amazon S3**, delivered over HTTPS through **Amazon CloudFront**.
2. **API call**: the browser calls **Amazon API Gateway**, which forwards the request to **AWS Lambda** (Python) for business logic.
3. **Direct upload**: Lambda returns a **presigned URL** so the browser uploads directly to **Amazon S3**, and writes metadata into **Amazon DynamoDB**.
4. **AI analysis**: after upload, Lambda calls the AI layer, **Rekognition** labels/moderates images, **Textract** extracts document text, **Polly** turns text into audio; labels and text are saved into **DynamoDB** for content-based search.
5. **Monitoring & security**: **Amazon CloudWatch** collects logs and metrics; an **IAM Role** grants least-privilege access to each service.

![InsightShare Architecture](/images/5-Workshop/5.1-Workshop-overview/insightshare_architecture.png)

#### AWS services used

| Service | Role in InsightShare | Reason for choosing |
|---|---|---|
| Amazon S3 | Stores uploaded files and hosts the static site | Durable and low cost, supports presigned URLs so the browser transfers files directly without going through Lambda |
| Amazon CloudFront | CDN delivering the static web interface over HTTPS | Faster global delivery and HTTPS for the frontend without managing a web server |
| Amazon API Gateway | Public API gateway for the frontend to call the back-end | Managed HTTP endpoint with throttling and CORS, no server to run |
| AWS Lambda | Back-end business logic in Python | No servers to manage and pay-per-invocation, scales automatically with load |
| Amazon DynamoDB | Stores file metadata with AI labels and extracted text | Serverless NoSQL with millisecond reads, fits the per-file metadata and search pattern |
| Amazon Rekognition | Labels images and moderates sensitive content | Ready-to-call AI, no model training, returns labels and moderation flags from a single API call |
| Amazon Textract | Extracts text from PDFs and scanned images | Ready-to-call OCR that makes documents searchable by their content |
| Amazon Polly | Turns extracted text into audio (mp3) | Ready-to-call text-to-speech, adds an accessibility feature with one API call |
| Amazon CloudWatch | Logging, metrics and alarms | Central monitoring for Lambda and API Gateway to debug and watch cost/usage |
| AWS IAM | Least-privilege access control | Grants each component only the permissions it needs, keeping files non-public |
