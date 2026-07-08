---
title: "Week 6 Worklog"
date: 2026-07-06
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 objectives

* Expose the API to the frontend via API Gateway.
* Complete the end-to-end flow on the cloud.

### Tasks during the week (06/07 - 10/07/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Create an HTTP API Gateway with Lambda proxy integration; define routes for each endpoint (POST /files, GET /files, GET /files/{id}). | 06/07/2026 | 06/07/2026 | [API Gateway lab](https://000079.awsstudygroup.com) |
| Tue | Configure CORS at the API Gateway layer (allowed origins, methods, headers) and add the Lambda invoke permission for the gateway. | 07/07/2026 | 07/07/2026 |  |
| Wed | Switch the frontend to call the API Gateway invoke URL instead of localhost. | 08/07/2026 | 08/07/2026 |  |
| Thu | Test the API with Postman and the frontend; fix CORS preflight, IAM permission and proxy mapping errors. | 09/07/2026 | 09/07/2026 | Postman |
| Fri | Review and finalize the full flow on the cloud: frontend → API Gateway → Lambda → presigned URL → direct upload to S3. | 10/07/2026 | 10/07/2026 |  |

### Results achieved

1. API Gateway provides a public endpoint the frontend can call with correct CORS.
2. The frontend now calls API Gateway instead of the local back-end.
3. Resolved common integration errors and completed the end-to-end cloud flow.
