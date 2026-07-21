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

**InsightShare** is a web application for uploading, analyzing and sharing images and documents, built entirely on a **serverless** architecture on AWS. Unlike plain storage, the moment a file is uploaded the system uses AWS AI services to understand its content, so users can search their files by what is inside them, not only by filename.

The build covers:
- File storage in **Amazon S3** with sharing through time-limited **presigned URLs**
- A serverless back-end on **AWS Lambda** (Python) behind **Amazon API Gateway**
- File metadata, AI labels and extracted text in **Amazon DynamoDB** for content-based search
- A content-understanding layer using **Amazon Rekognition**, **Amazon Textract** and **Amazon Bedrock** (Claude)
- The static frontend served over **Amazon CloudFront** (HTTPS), with monitoring in **Amazon CloudWatch**
- Least-privilege access through **AWS IAM** across every service

The project uses a serverless architecture built from:
- **Storage & sharing**: Amazon S3 for private file storage with presigned URLs; metadata in Amazon DynamoDB
- **Serverless back-end**: AWS Lambda (Python) behind Amazon API Gateway for all business logic
- **Content understanding with AI**: Amazon Rekognition (image labels), Amazon Textract (text extraction), Amazon Bedrock/Claude (Vietnamese Q&A and summary over documents), all ready-to-call with no model training
- **Smart search**: labels and extracted text indexed in DynamoDB to find files by content
- **Delivery & monitoring**: Amazon CloudFront for HTTPS delivery, Amazon CloudWatch for logs and metrics
- **Security**: AWS IAM roles following the least-privilege principle

#### Content

1. [Workshop overview](5.1-Workshop-overview/)
2. [Prerequisite](5.2-Prerequiste/)
3. [File storage with S3 + presigned URL](5.3-S3-storage/)
4. [Serverless back-end: Lambda + API Gateway + DynamoDB](5.4-serverless-backend/)
5. [Monitoring & Security (CloudWatch + IAM)](5.5-Policy/)
6. [Clean up](5.6-Cleanup/)
