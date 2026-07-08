---
title: "Week 10 Worklog"
date: 2026-08-03
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 objectives

* Automate deployment with CI/CD and run end-to-end testing.
* Measure, optimize performance and stabilize the system.

### Tasks during the week (03/08 - 07/08/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Set up CI/CD with GitHub Actions to run `sam build` and `sam deploy` on push to the main branch, authenticating to AWS via repository secrets. | 03/08/2026 | 03/08/2026 | [CI/CD lab](https://000084.awsstudygroup.com) |
| Tue | Run end-to-end and failure testing: large files, expired presigned links, wrong permissions and concurrent uploads. | 04/08/2026 | 04/08/2026 |  |
| Wed | Measure AI processing time (Rekognition/Textract/Polly) and search latency in CloudWatch; identify bottlenecks. | 05/08/2026 | 05/08/2026 |  |
| Thu | Optimize: increase Lambda memory for AI tasks, call AI asynchronously via S3 events so the upload request is not blocked. | 06/08/2026 | 06/08/2026 |  |
| Fri | Improve error handling and feedback (clearer upload/search messages); standardize the error codes returned by the API. | 07/08/2026 | 07/08/2026 |  |

### Results achieved

1. A CI/CD pipeline builds and deploys on push, cutting manual steps and deployment risk.
2. The system is well tested across scenarios, including concurrent load and failure cases.
3. Measured and optimized performance (Lambda memory, asynchronous AI processing), reducing latency and improving stability.
