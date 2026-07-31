---
title: "Overview"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### About InsightShare

**InsightShare** is a serverless web application for uploading, analyzing and sharing images and documents on AWS. It calls AWS AI services on each upload and stores what they return next to the file's metadata, and files are shared through time-limited links.

The platform is built on:
- **Amazon S3 + presigned URLs**: private file storage; the browser uploads and downloads directly through short-lived signed links.
- **AWS Lambda + Amazon API Gateway**: a Python back-end with no servers to manage, exposed as an HTTP API.
- **Amazon Cognito**: user sign-in, scoped by the JWT `sub` claim so each user sees only their own files.
- **Amazon DynamoDB**: file metadata with AI labels and extracted text, backing content-based search.
- **Amazon Rekognition, Textract and Bedrock**: the AI layer that labels images, extracts document text, and answers questions or summarizes a document in the same language as the question.
- **Amazon CloudFront + CloudWatch + IAM**: HTTPS delivery of the static frontend, monitoring, and least-privilege access control.

#### Architecture Overview

The numbered steps match the arrows in the architecture diagram, in order:

1. **User → CloudFront**: the browser loads the static web interface from S3, delivered over HTTPS through CloudFront.
2. **User → API Gateway → Lambda**: the browser calls API Gateway, which forwards the request to the Python Lambda for business logic.
3. **Lambda → signed URL**: Lambda returns a presigned URL, and the browser uses it to transfer the file directly with S3.
4. **Lambda → AI services**: Lambda calls Rekognition for image labels or Textract for document text, and the `ask` endpoint sends stored text to Bedrock to answer a question or summarize the document in the language of the question.
5. **Lambda → DynamoDB**: file metadata, AI labels and extracted text are written to DynamoDB, which backs content search.
6. **Monitoring and security**: CloudWatch collects logs and metrics; an IAM role grants least-privilege access to each service.

![InsightShare Architecture](/images/5-Workshop/5.1-Workshop-overview/insightshare_architecture-v6.png)

#### AWS services used

| Service | Role in InsightShare | Reason for choosing |
|---|---|---|
| Amazon S3 | Stores uploaded files and hosts the static site | Durable and low cost, supports presigned URLs so the browser transfers files directly without going through Lambda |
| Amazon CloudFront | CDN delivering the static web interface over HTTPS | Faster global delivery and HTTPS for the frontend without managing a web server |
| Amazon API Gateway | Public API gateway for the frontend to call the back-end; a JWT authorizer validates Cognito tokens | Managed HTTP endpoint with throttling and CORS, no server to run |
| Amazon Cognito | User sign-in; supplies the JWT whose `sub` claim scopes files per user | Managed user directory, callable straight from a static page, with no auth server to build |
| AWS Lambda | Back-end business logic in Python | No servers to manage and pay-per-invocation, scales automatically with load |
| Amazon DynamoDB | Stores file metadata with AI labels and extracted text | Serverless NoSQL with millisecond reads, fits the per-file metadata and search pattern |
| Amazon Rekognition | Labels images | Returns labels from a single API call, with no training step |
| Amazon Textract | Extracts text from PDFs and scanned images | Ready-to-call OCR that makes documents searchable by their content |
| Amazon Bedrock | Answers questions and summarizes a document in the same language as the question | Hosted Claude model; one API call turns the extracted text into an answer |
| Amazon CloudWatch | Logging, metrics and alarms | Central monitoring for Lambda and API Gateway to debug and watch cost/usage |
| AWS IAM | Least-privilege access control | Grants each component only the permissions it needs, keeping files non-public |