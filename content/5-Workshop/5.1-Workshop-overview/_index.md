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
- **Amazon Cognito**: user sign-in through the Hosted UI, so each user sees only their own files, scoped by the JWT `sub` claim.
- **Amazon DynamoDB**: file metadata with AI labels and extracted text, powering content-based search.
- **Amazon Rekognition, Textract and Bedrock (Claude)**: the AI layer, labels images, extracts document text, and answers questions or summarizes a document in Vietnamese, no model training.
- **Amazon CloudFront + CloudWatch + IAM**: HTTPS delivery of the static frontend, monitoring, and least-privilege access control.

#### Workshop Overview

InsightShare is built end to end across the following parts:

- A secure file storage layer on **Amazon S3** with **presigned URLs**.
- A back-end on **AWS Lambda** (Python) behind **Amazon API Gateway**, with metadata in **Amazon DynamoDB**.
- An **AI layer** that reads file content automatically: image labels with **Amazon Rekognition**, text extraction with **Amazon Textract**, and document Q&A / summary in Vietnamese with **Amazon Bedrock** (Claude).
- **Content search** over image labels and text extracted from documents.
- The static frontend on **Amazon S3 + CloudFront** (HTTPS).
- Monitoring and security with **Amazon CloudWatch** and a least-privilege **IAM Role**.

#### Architecture Overview

InsightShare follows a fully serverless flow. The numbered steps match the arrows in the architecture diagram, in order:

1. **User → CloudFront**: the browser loads the static web interface from **Amazon S3**, delivered over HTTPS through **Amazon CloudFront**.
2. **User → API Gateway → Lambda**: the browser calls **Amazon API Gateway**, which forwards the request to **AWS Lambda** (Python) for business logic.
3. **Lambda → S3 (presigned URL)**: Lambda returns a **presigned URL** so the browser uploads directly to **Amazon S3**.
4. **Lambda → AI services**: after upload, Lambda calls the AI layer, **Rekognition** labels images and **Textract** extracts document text, and the `ask` endpoint sends stored text to **Amazon Bedrock** (Claude) to answer questions or summarize the document in Vietnamese.
5. **Lambda → DynamoDB**: file metadata, AI labels and extracted text are written to **Amazon DynamoDB**, which powers content-based search.
6. **Monitoring & security**: **Amazon CloudWatch** collects logs and metrics; an **IAM Role** grants least-privilege access to each service.

![InsightShare Architecture](/images/5-Workshop/5.1-Workshop-overview/insightshare_architecture-v3.png)

#### AWS services used

| Service | Role in InsightShare | Reason for choosing |
|---|---|---|
| Amazon S3 | Stores uploaded files and hosts the static site | Durable and low cost, supports presigned URLs so the browser transfers files directly without going through Lambda |
| Amazon CloudFront | CDN delivering the static web interface over HTTPS | Faster global delivery and HTTPS for the frontend without managing a web server |
| Amazon API Gateway | Public API gateway for the frontend to call the back-end; a JWT authorizer validates Cognito tokens | Managed HTTP endpoint with throttling and CORS, no server to run |
| Amazon Cognito | User sign-in through the Hosted UI; supplies the JWT whose `sub` claim scopes files per user | Managed user directory and OAuth2 sign-in with no auth server to build |
| AWS Lambda | Back-end business logic in Python | No servers to manage and pay-per-invocation, scales automatically with load |
| Amazon DynamoDB | Stores file metadata with AI labels and extracted text | Serverless NoSQL with millisecond reads, fits the per-file metadata and search pattern |
| Amazon Rekognition | Labels images | Ready-to-call AI, no model training, returns labels from a single API call |
| Amazon Textract | Extracts text from PDFs and scanned images | Ready-to-call OCR that makes documents searchable by their content |
| Amazon Bedrock (Claude) | Answers questions and summarizes a document in Vietnamese | Hosted Claude model, no training; turns extracted text into direct answers with one API call |
| Amazon CloudWatch | Logging, metrics and alarms | Central monitoring for Lambda and API Gateway to debug and watch cost/usage |
| AWS IAM | Least-privilege access control | Grants each component only the permissions it needs, keeping files non-public |
