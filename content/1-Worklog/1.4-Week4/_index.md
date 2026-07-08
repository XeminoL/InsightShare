---
title: "Week 4 Worklog"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 objectives

* Move file storage to Amazon S3 with presigned URLs.
* Apply IAM least-privilege and configure CORS for the storage layer.

### Tasks during the week (22/06 - 26/06/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Create the private S3 bucket in ap-southeast-1, enable Block Public Access and default SSE-S3 encryption. | 22/06/2026 | 22/06/2026 | [Amazon S3](https://000057.awsstudygroup.com) |
| Tue | Write functions to generate presigned URLs for upload and download with boto3 (`generate_presigned_url`, methods put_object/get_object, expiry 900s); test with curl. | 23/06/2026 | 23/06/2026 |  |
| Wed | Configure the bucket CORS rule (allowed origins, PUT/GET methods, headers) so the browser can upload directly via the presigned URL. | 24/06/2026 | 24/06/2026 |  |
| Thu | Swap the local storage for S3 behind the storage interface from week 3, leaving the business logic untouched. | 25/06/2026 | 25/06/2026 |  |
| Fri | Write a least-privilege IAM policy scoping s3:PutObject/GetObject to this bucket ARN only; test upload/download and error cases (expired URL, wrong key). | 26/06/2026 | 26/06/2026 |  |

### Results achieved

1. Files are stored safely on S3 via presigned URLs while the bucket stays non-public.
2. Swapped storage to S3 with almost no business-logic change thanks to the interface.
3. Applied least-privilege early and tested the error cases.
