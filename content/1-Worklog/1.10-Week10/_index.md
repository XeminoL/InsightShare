---
title: "Week 10 Worklog"
date: 2026-08-03
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 objectives

* Make deploy and teardown repeatable, and run end-to-end testing.
* Measure, optimize performance and stabilize the system.

### Tasks during the week (03/08 - 07/08/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Script the deploy step (zip the function and run `aws lambda update-function-code`) so re-deploying after a code change is one command. | 03/08/2026 | 03/08/2026 |  |
| Tue | Run end-to-end and failure testing: large files, expired presigned links and wrong permissions. | 04/08/2026 | 04/08/2026 |  |
| Wed | Measure AI processing time (Rekognition/Textract/Bedrock) and search latency in CloudWatch; identify bottlenecks. | 05/08/2026 | 05/08/2026 |  |
| Thu | Optimize: increase Lambda memory for AI tasks, call AI asynchronously via S3 events so the upload request is not blocked. | 06/08/2026 | 06/08/2026 |  |
| Fri | Improve error handling and feedback (clearer upload/search messages); standardize the error codes returned by the API. | 07/08/2026 | 07/08/2026 |  |

### Results achieved

1. Deploy and teardown are scripted (zip + update-function-code; cleanup-aws.ps1), cutting manual steps and mistakes.
2. The system was tested across the main scenarios, including failure cases (large files, expired links, wrong permissions).
3. Measured and optimized performance (Lambda memory, asynchronous AI processing), reducing latency and improving stability.
