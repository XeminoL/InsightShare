---
title: "Serverless back-end: Lambda + API Gateway + DynamoDB"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Overview

This section covers the core of InsightShare: the serverless back-end and the frontend.

+ **Lambda (Python)** handles the business logic: receive requests, generate presigned URLs, write metadata.
+ **API Gateway** exposes the APIs the frontend calls.
+ **DynamoDB** stores the metadata of each file (name, timestamp, size, share link, AI labels, extracted text, etc.).
+ The **AI layer** (Rekognition, Textract, Polly) analyzes file content to label, moderate, extract text, and read audio.
+ The **frontend** (static web) is hosted on **S3 + CloudFront**.

After completing this section, the whole flow from browser → API Gateway → Lambda → S3/DynamoDB → AI layer will work end-to-end.

#### Content

- [Write the Lambda function](5.4.1-lambda/)
- [Create the API Gateway](5.4.2-api-gateway/)
- [Integrate DynamoDB metadata](5.4.3-dynamodb/)
- [Frontend S3 + CloudFront](5.4.4-frontend-cloudfront/)
- [Add AI: Rekognition, Textract, Polly](5.4.5-ai-services/)
