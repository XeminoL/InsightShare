---
title: "Proposal"
date: 2026-06-20
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# InsightShare
## A smart image & document sharing platform on AWS

### 1. Executive Summary

InsightShare is a web application for uploading, analyzing and sharing images/documents. After a file is uploaded, AWS AI services label images, extract document text, and answer questions or summarize a document in the same language as the question, so a file can be found from a word that appears inside it. It is fully **serverless** on AWS: no servers to manage, per-request billing, scaling with load. It uses S3, Lambda, API Gateway, DynamoDB, CloudFront and Cognito, with three AI services: Rekognition, Textract and Bedrock. Amazon Cognito handles sign-in, and the JWT `sub` claim scopes each file to its owner so users see only their own files.

### 2. Problem Statement

*The problem*
- Users need a fast way to store and share images/documents, but self-hosted solutions (an EC2 instance running 24/7) incur fixed costs even when idle and require manual operation, patching and scaling.
- As the number of files grows, finding the right file is hard because search is name-only.
- Even after finding a document, understanding what is inside still means opening and reading it.

*The solution*

InsightShare centralizes data and processing on a unified serverless stack:
- **Storage & sharing:** S3 stores files, shared via time-limited presigned URLs; metadata lives in DynamoDB.
- **Business logic:** Lambda + API Gateway generate presigned URLs, orchestrate AI analysis, and read/write data.
- **Content understanding with AI:** Rekognition labels images, Textract extracts document text, and Bedrock answers questions and summarizes documents in the same language as the question. All three are called through the API with no training step.
- **Smart search:** labels and extracted text are stored in DynamoDB to find files by content.

*Benefits*
- The serverless model bills per use; at demo scale the total stays under $1/month.
- User files are non-public and reachable only through expiring signed links; the only public bucket is the one holding the static web page. Permissions follow IAM least-privilege and CloudWatch monitors the system.
- Search runs over the AI labels and the extracted text, so a file turns up from a word that appears inside it.

### 3. Solution Architecture

*Overview*
The browser loads the static frontend from **S3 + CloudFront** over HTTPS, signs in through **Cognito**, then calls **API Gateway**, which forwards to **Lambda**. API Gateway runs a JWT authorizer that validates the Cognito token, and Lambda reads the `sub` claim to scope every file to its owner. Lambda generates presigned URLs so the browser transfers files directly with S3. After upload, Lambda calls Rekognition, Textract or Bedrock and stores the results in DynamoDB for search. CloudWatch collects logs and metrics; IAM enforces least-privilege access.

![InsightShare Architecture](/images/2-Proposal/insightshare_architecture-v6.png)

*AWS Services Used*

| Service | Function |
|---|---|
| Amazon S3 | Store user files; host the static web frontend |
| Amazon CloudFront | CDN for the web, HTTPS, faster delivery |
| Amazon API Gateway | Public API gateway for the application; a JWT authorizer validates Cognito tokens |
| Amazon Cognito | User sign-in; the JWT `sub` claim scopes each file to its owner for per-user isolation |
| AWS Lambda | Business logic; a single handler routes API Gateway HTTP API requests by method and path |
| Amazon DynamoDB | Store metadata + AI labels + extracted text for search |
| Amazon Rekognition | Image labeling |
| Amazon Textract | Text extraction from PDF/scanned images |
| Amazon Bedrock | Document Q&A and summary, answered in the question's language |
| Amazon CloudWatch | Logs, metrics, alarms for monitoring |
| AWS IAM | Least-privilege access for Lambda and each AI service |

*Component Design*
- **Frontend:** a static web page to pick files, show the list with AI labels, a content search box, and a box to ask a question about a document.
- **API:** endpoints to request an upload URL, run the AI analysis, list/search files, get a download URL, ask a question about one document, and ask a question across the whole library.

### 4. Technical Implementation

*Implementation Phases*

| Phase | Description |
|---|---|
| 1. Foundations & design | Study AWS fundamentals, finalize the topic, draw the architecture, set up the account securely, design the DynamoDB schema. |
| 2. Basic application | Build the web app locally, separate the storage and AI layers. |
| 3. Move to the cloud | S3 + presigned URLs, Lambda + API Gateway, DynamoDB integration and presigned download links. |
| 4. AI layer & search | Integrate Rekognition/Textract/Bedrock, store results in DynamoDB, build smart search. |
| 5. Finalize & operate | CloudFront + HTTPS, CloudWatch logs/alarms, cost/security optimization, a repeatable deploy/cleanup script. |

*Technical Requirements*
- An AWS account, region `ap-southeast-1`.
- Tools: AWS CLI, Python 3, boto3.
- Knowledge: S3, Lambda, API Gateway, DynamoDB, IAM, CloudWatch and AI services.

### 5. Budget Estimation

At demo scale the pay-per-use serverless model and Free Tier keep cost under $1/month. An AWS Budget `InsightShare-Monthly` with a $5/month limit tracks spend. Detailed figures use the [AWS Pricing Calculator](https://calculator.aws/).

At real scale the cost grows with usage, dominated by the AI services. A rough monthly estimate for **1,000 active users**:

| Service | Basis | Est./month |
| --- | --- | --- |
| AWS Lambda | ~120k invocations | ~$0.10 |
| Amazon API Gateway | ~120k HTTP requests | ~$0.12 |
| Amazon S3 | ~40 GB stored + requests | ~$1.00 |
| Amazon DynamoDB | ~140k read/write | ~$0.20 |
| Amazon Rekognition | 20k images | ~$20.00 |
| Amazon Bedrock | ~20k summaries + Q&A | ~$15.00 |
| Amazon CloudFront | ~30 GB out | ~$2.50 |
| **Total** | | **~$40/month** |

### 6. Risk Assessment

| Risk | Impact | Probability | Mitigation |
|---|---|---|---|
| Misconfigured IAM/policy causing access errors | Medium | Medium | Least-privilege, careful testing before broadening permissions |
| Unexpected cost | Low | Low | Budget Alert, limit file size/type sent to AI, clean up after testing |
| Large files causing Lambda/API Gateway timeouts | Medium | Low | Presigned URLs upload directly to S3, so the file body never passes through Lambda or API Gateway |
| AI services slow or unavailable | Low | Medium | Analysis is a separate call from upload and each AI call fails soft, so a file stays listed and downloadable even when analysis does not complete |

*Contingency plan:* keep a scripted teardown to remove all resources quickly.

### 7. Expected Outcomes

*Technical Improvements*
- A working end-to-end web application: upload → automatic AI content analysis → list → content-based search → ask a question about a document → download via presigned link.
- Serverless architecture combined with AWS managed AI services.
- Document Q&A implemented with Amazon Bedrock: the `ask` endpoint takes a document and a question, and is wired to the `bedrock:InvokeModel` call with the inference-profile model id and full request/response handling.

*Long-term Value*
- An extensible platform: orchestrate a multi-step AI pipeline, trigger analysis from S3 events instead of a client call, support more file types.
- Detailed workshop documentation so others can follow and extend the project.