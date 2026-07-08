---
title: "Week 2 Worklog"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 objectives

* Design the InsightShare architecture and write the Proposal.
* Set up the AWS account securely to start working.
* Study presigned URLs and the AI services to be used.

### Tasks during the week (08/06 - 12/06/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Study IAM: users, groups, roles, the difference between identity and resource policies, and the least-privilege principle to apply per service later. | 08/06/2026 | 08/06/2026 | [AWS IAM](https://000002.awsstudygroup.com) |
| Tue | Draw the InsightShare serverless architecture in draw.io: browser → CloudFront/S3 → API Gateway → Lambda → S3/DynamoDB and the Rekognition/Textract/Polly branch. | 09/06/2026 | 09/06/2026 | [draw.io](https://draw.io/) |
| Wed | Write the Proposal: problem, solution, the AWS services table (10 services) and the rationale for each, all in region ap-southeast-1. | 10/06/2026 | 10/06/2026 |  |
| Thu | Create the AWS account, enable root MFA, create a dedicated IAM user with admin off-root, configure the AWS CLI with `aws configure`, and set a monthly Budget alert. | 11/06/2026 | 11/06/2026 |  |
| Fri | Design the DynamoDB metadata table (partition key fileId, attributes for owner, timestamp, AI labels and extracted text); study presigned URLs and the AI APIs detect_labels, detect_document_text and synthesize_speech. | 12/06/2026 | 12/06/2026 | [AI services](https://000056.awsstudygroup.com) |

### Results achieved

1. Produced the architecture diagram and a complete Proposal explaining each choice.
2. Set up the AWS account to security standards: MFA, dedicated IAM user, Budgets, CLI.
3. Understood presigned URLs and how the AI services will be called.
