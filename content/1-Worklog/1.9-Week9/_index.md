---
title: "Week 9 Worklog"
date: 2026-07-27
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 objectives

* Build smart search by labels and content.
* Move the frontend to CloudFront; add monitoring and optimization.

### Tasks during the week (27/07 - 31/07/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Build file search over the Rekognition labels and Textract text in DynamoDB, matching a keyword against the stored labels and text attributes. | 27/07/2026 | 27/07/2026 |  |
| Tue | Host the frontend on S3 + CloudFront with HTTPS and Origin Access Control, so the bucket stays private behind the distribution. | 28/07/2026 | 28/07/2026 | [CloudFront](https://cloudjourney.awsstudygroup.com/1-explore/) |
| Wed | Have Lambda log to CloudWatch and create a metric alarm on the function Errors metric. | 29/07/2026 | 29/07/2026 |  |
| Thu | Optimize cost (an S3 lifecycle rule for old objects, CloudFront cache TTL) and review system-wide security and least-privilege. | 30/07/2026 | 30/07/2026 |  |
| Fri | Optimize cost and write a repeatable cleanup script (cleanup-aws.ps1) that tears the whole stack down. | 31/07/2026 | 31/07/2026 |  |

### Results achieved

1. Files can be found by content (image labels or document text), not just by name.
2. The frontend runs over CloudFront with HTTPS; the system has CloudWatch logs and alarms.
3. Reduced cost, tightened security, and scripted the teardown of every resource.
