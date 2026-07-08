---
title: "Week 5 Worklog"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 objectives

* Move the back-end to AWS Lambda, fully serverless.
* Attach an IAM Role and test Lambda independently.

### Tasks during the week (29/06 - 03/07/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Write a plain Lambda handler that routes API Gateway HTTP API events by method and path, and package it into a deployment zip for the Python runtime. | 29/06/2026 | 29/06/2026 | [Serverless lab](https://000078.awsstudygroup.com) |
| Tue | Handle cold start and tune the function: raise memory (e.g. 512 MB), set a sensible timeout (15s) and read config from environment variables. | 30/06/2026 | 30/06/2026 |  |
| Wed | Create an execution IAM Role for Lambda with AWSLambdaBasicExecutionRole plus a least-privilege inline policy scoped to the needed bucket. | 01/07/2026 | 01/07/2026 | [IAM Role](https://000048.awsstudygroup.com) |
| Thu | Separate environment variables for local and cloud (bucket name, region, table name) so the same code runs in both. | 02/07/2026 | 02/07/2026 |  |
| Fri | Test the Lambda via direct `invoke` with sample event payloads and read the CloudWatch logs before wiring the API. | 03/07/2026 | 03/07/2026 |  |

### Results achieved

1. The back-end runs fully serverless on Lambda, with cold start handled and resources tuned.
2. Lambda has its own least-privilege IAM Role instead of broad permissions.
3. The Lambda function passed independent testing before integration.
