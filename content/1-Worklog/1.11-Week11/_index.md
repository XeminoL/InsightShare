---
title: "Week 11 Worklog"
date: 2026-08-10
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Week 11 objectives

* Add advanced features and polish the user experience.
* Study more related AWS services to broaden knowledge.

### Tasks during the week (10/08 - 14/08/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Add delete-file (remove the S3 object and DynamoDB item together), expiring share links and validation of file size/type before upload. | 10/08/2026 | 10/08/2026 |  |
| Tue | Improve search: filter by Rekognition labels and show results with image thumbnails (presigned GET URLs). | 11/08/2026 | 11/08/2026 |  |
| Wed | Study Amazon Cognito (user pool, hosted UI, JWT) to prepare user sign-in for the API. | 12/08/2026 | 12/08/2026 | [Cognito lab](https://000081.awsstudygroup.com) |
| Thu | Study Step Functions and how a state machine could orchestrate the multi-step Rekognition → Textract → Polly pipeline. | 13/08/2026 | 13/08/2026 | [Modernize](https://cloudjourney.awsstudygroup.com/4-modernize/) |
| Fri | Measure and tune performance: AI processing time per file and search latency, reading the durations from CloudWatch. | 14/08/2026 | 14/08/2026 |  |

### Results achieved

1. Added advanced features (delete file, expiring links, upload limits) as a personal contribution.
2. The search page is easier to use with label filters and thumbnails.
3. Broadened knowledge of Cognito and Step Functions, with directions to extend the system.
