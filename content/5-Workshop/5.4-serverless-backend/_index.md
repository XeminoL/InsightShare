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
+ **API Gateway** exposes the APIs the frontend calls, with a JWT authorizer for Cognito tokens.
+ **Amazon Cognito** signs users in through the Hosted UI, so each user sees only their own files.
+ **DynamoDB** stores the metadata of each file (name, timestamp, size, AI labels, extracted text, etc.).
+ The **AI layer** (Rekognition, Textract, Bedrock/Claude) labels images, extracts document text, and answers questions about a document.
+ The **frontend** (static web) is hosted on **S3 + CloudFront**.

After completing this section, the whole flow from browser → API Gateway → Lambda → S3/DynamoDB → AI layer will work end-to-end.

#### Content

- [Write the Lambda function](5.4.1-lambda/)
- [Create the API Gateway](5.4.2-api-gateway/)
- [Integrate DynamoDB metadata](5.4.3-dynamodb/)
- [Frontend S3 + CloudFront](5.4.4-frontend-cloudfront/)
- [Per-user sign-in with Amazon Cognito](5.4.5-cognito-auth/)
- [Add AI: Rekognition, Textract, Bedrock (Claude)](5.4.6-ai-services/)
