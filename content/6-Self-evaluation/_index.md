---
title: "Self-Assessment"
date: 2026-06-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

The internship at **Amazon Web Services Vietnam Co., Ltd.** (First Cloud AI Journey program) ran from **June 1, 2026** to **August 15, 2026**.

The main deliverable is **InsightShare**, a serverless web application for uploading and sharing images and documents on AWS. The work involved S3, Lambda, API Gateway, DynamoDB, CloudFront, IAM and CloudWatch; Python (boto3); and bilingual technical documentation.

The table below is a self-assessment against eight criteria, one rating each: Good / Fair.

| No. | Criteria | Good | Fair | Comment |
| --- | --- | --- | --- | --- |
| 1 | **Professional knowledge & skills** | x | | Applied core AWS services in one end-to-end project. |
| 2 | **Ability to learn** | x | | Picked up new services (Lambda, presigned URLs, CloudFront) from docs and labs. |
| 3 | **Proactiveness** | x | | Took initiative on the design and self-studied the services needed. |
| 4 | **Discipline** | | x | Kept a fixed office schedule and delivered the weekly worklog on time. |
| 5 | **Communication** | x | | Reported work clearly in writing and in bilingual technical documentation. |
| 6 | **Teamwork** | | x | Integrated with the team, asked mentors when blocked, and helped when needed. |
| 7 | **Problem-solving** | x | | Diagnosed integration issues (CORS, IAM, cold start, S3 307, DynamoDB `Decimal`) from CloudWatch logs. |
| 8 | **Contribution to the project** | x | | Implemented InsightShare end-to-end and added extra features (delete, expiring links, upload limits). |

### Lessons Learned

* Presigned URL behaviour differs across S3 regions: the default global endpoint returns HTTP 307 for a Singapore bucket, so the client must pin the regional endpoint and use Signature V4 to get a working URL.
* A running Lambda caches its execution credentials, so an IAM policy change takes effect only after the function is updated or redeployed to spin up a fresh execution environment.
* `text` is a DynamoDB reserved word, so an update expression that touches it needs `ExpressionAttributeNames` to alias the field.
* DynamoDB returns numbers as Python `Decimal`, which `json.dumps` cannot serialize; a custom JSON encoder is needed on every API response.
* AI calls (Textract, Bedrock) should fail soft and stay wrapped, so one unavailable service degrades gracefully instead of breaking the whole upload and analysis flow.
* CloudWatch logs are the primary tool for diagnosing serverless runtime errors, since there is no server to attach to.
