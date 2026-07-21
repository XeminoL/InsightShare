---
title: "Workshop"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
alwaysopen: false
---

# InsightShare: Building a Serverless Image & Document Sharing Platform with AI on AWS

#### Overview

**InsightShare** is a web application for uploading, analyzing and sharing images and documents, built entirely on a **serverless** architecture on AWS. When a file is uploaded, the system uses AWS AI services to read its content, so files can be searched by what is inside them, not only by filename.

The build covers:
- **Amazon S3** for private file storage, shared through time-limited **presigned URLs**
- A serverless back-end on **AWS Lambda** (Python) behind **Amazon API Gateway**
- File metadata, AI labels and extracted text in **Amazon DynamoDB**, which back content-based search
- A content-understanding layer with **Amazon Rekognition**, **Amazon Textract** and **Amazon Bedrock** (Claude), all ready-to-call with no model training
- The static frontend served over **Amazon CloudFront** (HTTPS), with monitoring in **Amazon CloudWatch**
- Least-privilege access through **AWS IAM** across every service

#### Content

1. [Workshop overview](5.1-Workshop-overview/)
2. [Prerequisite](5.2-Prerequiste/)
3. [File storage with S3 + presigned URL](5.3-S3-storage/)
4. [Serverless back-end: Lambda + API Gateway + DynamoDB](5.4-serverless-backend/)
5. [Monitoring & Security (CloudWatch + IAM)](5.5-Policy/)
6. [Clean up](5.6-Cleanup/)
