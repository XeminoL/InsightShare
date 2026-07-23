---
title: "References"
date: 2026-07-22
weight: 8
chapter: false
pre: " <b> 8. </b> "
---

### Project links

| Resource | Link |
| --- | --- |
| Source code (GitHub repository) | [github.com/XeminoL/InsightShare](https://github.com/XeminoL/InsightShare) |
| Live application (demo) | [insightshare.dangthaikhang34.workers.dev](https://insightshare.dangthaikhang34.workers.dev) |

### What I read while building

- AWS Study Group labs (cloudjourney.awsstudygroup.com): the S3, serverless back-end and AI-services labs, which I followed for the account setup and the first working versions of each layer.
- boto3 reference for the calls I used: `generate_presigned_url`, DynamoDB `put_item`/`update_item`/`scan`, `detect_labels`, `detect_document_text` and Bedrock `invoke_model`.
- AWS docs I checked when something did not work: the S3 presigned-URL page (for the 307 / SigV4 issue) and the DynamoDB reserved-words list (for the `text` attribute).
