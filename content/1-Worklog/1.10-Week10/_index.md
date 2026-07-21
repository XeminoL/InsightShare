---
title: "Week 10 Worklog"
date: 2026-08-03
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 objectives

* Search files by content and move the frontend to S3 + CloudFront.
* Add CloudWatch monitoring and reduce cost.

### Tasks during the week (03/08 - 07/08/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Build file search over the Rekognition labels and Textract text in DynamoDB, matching a keyword against the stored label and text attributes for the owner. | 03/08/2026 | 03/08/2026 |  |
| Tue | Add an `/ask` endpoint that answers over the whole library and cites which file each answer came from. | 04/08/2026 | 04/08/2026 |  |
| Wed | Host the frontend on S3 + CloudFront with HTTPS and Origin Access Control, so the file bucket stays private behind the distribution. | 05/08/2026 | 05/08/2026 | [CloudFront](https://cloudjourney.awsstudygroup.com/1-explore/) |
| Thu | Have Lambda log to CloudWatch and create a metric alarm on the function Errors metric. | 06/08/2026 | 06/08/2026 |  |
| Fri | Optimize cost (an S3 lifecycle rule for old objects, CloudFront cache TTL) and review least-privilege across the system. | 07/08/2026 | 07/08/2026 |  |

### Results achieved

1. Files can be found by content (image labels or document text), and the `/ask` endpoint answers over the library with the source file cited.
2. The frontend runs on S3 + CloudFront over HTTPS with Origin Access Control; the file bucket stays private.
3. Lambda logs and an Errors alarm are in CloudWatch; cost is trimmed with S3 lifecycle and CloudFront TTL, and permissions were reviewed for least-privilege.
