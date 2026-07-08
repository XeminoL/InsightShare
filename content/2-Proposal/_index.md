---
title: "Proposal"
date: 2026-06-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# InsightShare
## A smart image & document sharing platform on AWS (serverless + AI)

### 1. Executive Summary

InsightShare is a web application for uploading, analyzing and sharing images/documents. On upload, AWS AI services understand the content: labeling images, moderating sensitive content, extracting document text and turning text into audio, so users search by content rather than only by name. It is fully **serverless** on AWS (region `ap-southeast-1`): no servers to manage, per-request billing so idle cost stays minimal, scaling with load. It uses S3, Lambda, API Gateway, DynamoDB and CloudFront with three AI services: Rekognition, Textract and Polly.

### 2. Problem Statement

*The problem*
- Users need a fast way to store and share images/documents, but self-hosted solutions (an EC2 instance running 24/7) incur fixed costs even when idle and require manual operation, patching and scaling.
- As the number of files grows, finding the right file is hard because search is name-only.
- There is no automatic content moderation, so unsuitable files can slip through.

*The solution*
InsightShare centralizes data and processing on a unified serverless stack:
- **Storage & sharing:** S3 stores files (private bucket), shared via time-limited presigned URLs; metadata lives in DynamoDB.
- **Business logic:** Lambda + API Gateway generate presigned URLs, orchestrate AI analysis, and read/write data.
- **Content understanding with AI:** Rekognition labels and moderates images, Textract extracts document text, Polly turns text into audio. All are ready-to-call services, with no model training.
- **Smart search:** labels and extracted text are stored in DynamoDB to find files by content.

*Benefits & ROI*
- **Centralization:** storage, sharing and analysis on a single platform, reducing manual steps.
- **Smart:** auto-labeling, text extraction, moderation and content-based search on top of storage.
- **Low cost:** the serverless model bills per use; the AI services stay within the Free Tier at demo scale, for a total under $1/month.
- **Reliable & secure:** files are non-public, IAM least-privilege, monitored with CloudWatch.
- A practical learning platform on serverless + AI architecture, extensible into a larger product.

### 3. Solution Architecture

*Overview*
The browser loads the static frontend from **S3 + CloudFront (HTTPS)** → calls **API Gateway** → **Lambda (Python)**. Lambda generates presigned URLs so the browser uploads/downloads directly to **S3**. After upload, Lambda calls the AI services (**Rekognition / Textract / Polly**) and stores results in **DynamoDB** for search. **CloudWatch** monitors logs/metrics; **IAM** enforces least-privilege access.

![InsightShare Architecture](/images/2-Proposal/insightshare_architecture.png)

*AWS Services Used*

| Service | Function |
|---|---|
| Amazon S3 | Store user files & Polly-generated audio; host the static web frontend |
| Amazon CloudFront | CDN for the web, HTTPS, faster delivery |
| Amazon API Gateway | Public API gateway for the application |
| AWS Lambda | Business logic (Python/boto3); a single handler routes API Gateway HTTP API requests by method and path |
| Amazon DynamoDB | Store metadata + AI labels + extracted text for search |
| Amazon Rekognition | Image labeling (DetectLabels) + moderation (DetectModerationLabels) |
| Amazon Textract | Text extraction from PDF/scanned images (DetectDocumentText) |
| Amazon Polly | Text-to-speech audio (SynthesizeSpeech) |
| Amazon CloudWatch | Logs, metrics, alarms for monitoring |
| AWS IAM | Least-privilege access for Lambda and each AI service |

*Component Design*
- **Frontend:** a static web page (HTML/JS) to pick files, show the list with AI labels, a content search box, and get share links.
- **API:** endpoints to request an upload URL, confirm upload (triggers AI analysis), list/search files, and get a download URL.
- **Storage:** files & audio in S3; metadata + AI results in DynamoDB.
- **Security:** least-privilege IAM Role; files are non-public, accessible only via presigned URLs.

### 4. Technical Implementation

*Implementation Phases*

| Phase | Description |
|---|---|
| 1. Foundations & design | Study AWS fundamentals, finalize the topic, draw the architecture, set up the account securely (MFA, IAM user, CLI, Budgets), design the DynamoDB schema. |
| 2. Basic application | Build the web app (FastAPI + frontend) locally, separate the storage and AI layers. |
| 3. Move to the cloud | S3 + presigned URLs, Lambda + API Gateway, DynamoDB integration and the share-link feature. |
| 4. AI layer & search | Integrate Rekognition/Textract/Polly, store results in DynamoDB, build smart search. |
| 5. Finalize & operate | CloudFront + HTTPS, CloudWatch logs/alarms, cost/security optimization, IaC with AWS SAM, CI/CD. |

*Technical Requirements*
- An AWS account (Free Tier), region `ap-southeast-1` (Singapore).
- Tools: AWS CLI, Python 3, boto3, (optional) AWS SAM.
- Knowledge: S3, Lambda, API Gateway, DynamoDB, IAM, CloudWatch and AI services (Rekognition, Textract, Polly).

### 5. Timeline & Milestones

| Milestone | Activity |
|---|---|
| Preparation | Finalize the architecture, design DynamoDB, set up the AWS account, write the Proposal |
| Core development | S3 storage + presigned URLs, Lambda + API Gateway back-end, DynamoDB + share link |
| AI integration | Rekognition + Textract + Polly; smart content-based search |
| Finalization | CloudFront, CloudWatch, cost/security optimization, SAM, CI/CD |
| Operation | Monitoring, testing, security review and resource clean-up |

Key milestones: complete the sharing MVP (upload + link) first; add the AI layer and search next; then finalize, optimize and operate.

### 6. Budget Estimation

Estimated cost is low with the pay-per-use serverless model, and the AI services stay within the Free Tier at demo scale. Detailed figures will be computed with the [AWS Pricing Calculator](https://calculator.aws/).

| Service | Est./month |
| --- | --- |
| AWS Lambda | ~$0.00 (within Free Tier) |
| Amazon API Gateway | ~$0.01 |
| Amazon S3 (storage + requests) | ~$0.10-0.20 |
| Amazon DynamoDB (on-demand) | ~$0.00-0.05 |
| Amazon CloudFront | ~$0.00-0.10 |
| Rekognition + Textract + Polly | ~$0.00 (demo scale, within Free Tier) |
| **Total** | **< $1/month (demo level)** |

Measured against the running infrastructure, the actual operating cost for July 2026 was essentially $0: every service sits within the Free Tier or on a pay-per-use model, so an idle stack incurs almost nothing. An AWS Budget `InsightShare-Monthly` with a $5/month limit was created to track spend, in addition to the account's existing zero-spend budget.

At real scale the cost grows with usage. A rough monthly estimate for **1,000 active users** (about 20,000 uploads, each analyzed once, plus browsing and search):

| Service | Basis | Est./month |
| --- | --- | --- |
| AWS Lambda | ~120k invocations | ~$0.10 |
| Amazon API Gateway | ~120k HTTP requests | ~$0.12 |
| Amazon S3 | ~40 GB stored + requests | ~$1.00 |
| Amazon DynamoDB (on-demand) | ~140k read/write | ~$0.20 |
| Amazon Rekognition | 20k images (DetectLabels) | ~$20.00 |
| Amazon Polly | ~5M characters (neural) | ~$80.00 |
| Amazon CloudFront | ~30 GB out | ~$2.50 |
| **Total** | | **~$105/month** |

The AI services (Rekognition, Polly) dominate the cost at scale, which is the trade-off for not training or hosting models. The demo stays near $0 because volumes are within the Free Tier.

### 7. Risk Assessment

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Misconfigured IAM/policy causing access errors | Medium | Medium | Least-privilege, careful testing before broadening permissions |
| Unexpected cost (many AI calls) | Low | Low | Budget Alert, limit file size/type sent to AI, clean up after testing |
| Large files causing Lambda/API Gateway timeouts | Medium | Low | Presigned URLs upload directly to S3; call AI asynchronously via S3 events |
| AI services slow or results not yet accurate | Low | Medium | Asynchronous AI analysis, so files remain shareable even before analysis completes |

*Contingency plan:* use Infrastructure-as-Code (AWS SAM) to recreate/delete infrastructure quickly.

### 8. Expected Outcomes

*Technical Improvements*
- A working end-to-end web application: upload → automatic AI content analysis → list → content-based search → share via link.
- Mastery of serverless architecture combined with AWS managed AI services.

*Long-term Value*
- An extensible platform: add user sign-in (Cognito), orchestrate a multi-step AI pipeline (Step Functions), support more file types.
- Detailed workshop documentation so others can follow and extend the project.
