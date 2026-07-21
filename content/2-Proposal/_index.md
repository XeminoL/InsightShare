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

InsightShare is a web application for uploading, analyzing and sharing images/documents. On upload, AWS AI services label images, extract document text, and answer questions or summarize a document in Vietnamese, so files can be found by content and not only by name. It is fully **serverless** on AWS (region `ap-southeast-1`): no servers to manage, per-request billing, scaling with load. It uses S3, Lambda, API Gateway, DynamoDB, CloudFront and Cognito, with three AI services: Rekognition, Textract and Bedrock (Claude). Amazon Cognito handles sign-in, and the JWT `sub` claim scopes each file to its owner so users see only their own files.

### 2. Problem Statement

*The problem*
- Users need a fast way to store and share images/documents, but self-hosted solutions (an EC2 instance running 24/7) incur fixed costs even when idle and require manual operation, patching and scaling.
- As the number of files grows, finding the right file is hard because search is name-only.
- Even after finding a document, understanding what is inside still means opening and reading it.

*The solution*

InsightShare centralizes data and processing on a unified serverless stack:
- **Storage & sharing:** S3 stores files (private bucket), shared via time-limited presigned URLs; metadata lives in DynamoDB.
- **Business logic:** Lambda + API Gateway generate presigned URLs, orchestrate AI analysis, and read/write data.
- **Content understanding with AI:** Rekognition labels images, Textract extracts document text, and Bedrock (a Claude model) answers questions and summarizes documents in Vietnamese. All are ready-to-call services, with no model training.
- **Smart search:** labels and extracted text are stored in DynamoDB to find files by content.

*Benefits*
- The serverless model bills per use; at demo scale the total stays under $1/month.
- Files are non-public, permissions follow IAM least-privilege, and CloudWatch monitors the system.
- AI labels and extracted text let files be found by what is inside them, not only by name.

### 3. Solution Architecture

*Overview*
The browser loads the static frontend from **S3 + CloudFront (HTTPS)** → signs in through **Amazon Cognito** → calls **API Gateway** → **Lambda (Python)**. API Gateway runs a JWT authorizer that validates the Cognito token, and Lambda reads the `sub` claim to scope every file to its owner. Lambda generates presigned URLs so the browser uploads/downloads directly to **S3**. After upload, Lambda calls the AI services (**Rekognition / Textract / Bedrock**) and stores results in **DynamoDB** for search. **CloudWatch** monitors logs/metrics; **IAM** enforces least-privilege access.

![InsightShare Architecture](/images/2-Proposal/insightshare_architecture-v6.png)

*AWS Services Used*

| Service | Function |
|---|---|
| Amazon S3 | Store user files; host the static web frontend |
| Amazon CloudFront | CDN for the web, HTTPS, faster delivery |
| Amazon API Gateway | Public API gateway for the application; a JWT authorizer validates Cognito tokens |
| Amazon Cognito | User sign-in (Hosted UI); the JWT `sub` claim scopes each file to its owner for per-user isolation |
| AWS Lambda | Business logic (Python/boto3); a single handler routes API Gateway HTTP API requests by method and path |
| Amazon DynamoDB | Store metadata + AI labels + extracted text for search |
| Amazon Rekognition | Image labeling (DetectLabels) |
| Amazon Textract | Text extraction from PDF/scanned images (DetectDocumentText) |
| Amazon Bedrock (Claude) | Vietnamese Q&A and document summary (InvokeModel) |
| Amazon CloudWatch | Logs, metrics, alarms for monitoring |
| AWS IAM | Least-privilege access for Lambda and each AI service |

*Component Design*
- **Frontend:** a static web page (HTML/JS) to pick files, show the list with AI labels, a content search box, and a box to ask a question about a document.
- **API:** endpoints to request an upload URL, confirm upload (triggers AI analysis), list/search files, get a download URL, and ask a question about a document.

### 4. Technical Implementation

*Implementation Phases*

| Phase | Description |
|---|---|
| 1. Foundations & design | Study AWS fundamentals, finalize the topic, draw the architecture, set up the account securely (MFA, IAM user, CLI, Budgets), design the DynamoDB schema. |
| 2. Basic application | Build the web app (FastAPI + frontend) locally, separate the storage and AI layers. |
| 3. Move to the cloud | S3 + presigned URLs, Lambda + API Gateway, DynamoDB integration and presigned download links. |
| 4. AI layer & search | Integrate Rekognition/Textract/Bedrock, store results in DynamoDB, build smart search. |
| 5. Finalize & operate | CloudFront + HTTPS, CloudWatch logs/alarms, cost/security optimization, a repeatable deploy/cleanup script. |

*Technical Requirements*
- An AWS account (Free Tier), region `ap-southeast-1` (Singapore).
- Tools: AWS CLI, Python 3, boto3.
- Knowledge: S3, Lambda, API Gateway, DynamoDB, IAM, CloudWatch and AI services (Rekognition, Textract, Bedrock).

### 5. Budget Estimation

At demo scale the pay-per-use serverless model and Free Tier keep cost under $1/month. An AWS Budget `InsightShare-Monthly` with a $5/month limit tracks spend. Detailed figures use the [AWS Pricing Calculator](https://calculator.aws/).

At real scale the cost grows with usage, dominated by the AI services (the trade-off for not training or hosting models). A rough monthly estimate for **1,000 active users** (about 20,000 uploads, each analyzed once, plus browsing and search):

| Service | Basis | Est./month |
| --- | --- | --- |
| AWS Lambda | ~120k invocations | ~$0.10 |
| Amazon API Gateway | ~120k HTTP requests | ~$0.12 |
| Amazon S3 | ~40 GB stored + requests | ~$1.00 |
| Amazon DynamoDB (on-demand) | ~140k read/write | ~$0.20 |
| Amazon Rekognition | 20k images (DetectLabels) | ~$20.00 |
| Amazon Bedrock (Claude Haiku) | ~20k summaries + Q&A | ~$15.00 |
| Amazon CloudFront | ~30 GB out | ~$2.50 |
| **Total** | | **~$40/month** |

### 6. Risk Assessment

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Misconfigured IAM/policy causing access errors | Medium | Medium | Least-privilege, careful testing before broadening permissions |
| Unexpected cost (many AI calls) | Low | Low | Budget Alert, limit file size/type sent to AI, clean up after testing |
| Large files causing Lambda/API Gateway timeouts | Medium | Low | Presigned URLs upload directly to S3; call AI asynchronously via S3 events |
| AI services slow or results not yet accurate | Low | Medium | Asynchronous AI analysis, so files remain downloadable even before analysis completes |

*Contingency plan:* keep a scripted teardown (cleanup-aws.ps1) to remove all resources quickly.

### 7. Expected Outcomes

*Technical Improvements*
- A working end-to-end web application: upload → automatic AI content analysis → list → content-based search → ask a question about a document → download via presigned link.
- Serverless architecture combined with AWS managed AI services.
- Document Q&A implemented with Amazon Bedrock (Claude): the `ask` endpoint takes a document and a question, and is wired to the `bedrock:InvokeModel` call with the inference-profile model id and full request/response handling.

*Long-term Value*
- An extensible platform: add user sign-in (Cognito), orchestrate a multi-step AI pipeline (Step Functions), support more file types.
- Detailed workshop documentation so others can follow and extend the project.
