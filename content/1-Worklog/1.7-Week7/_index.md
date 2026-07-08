---
title: "Week 7 Worklog"
date: 2026-07-13
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 objectives

* Integrate DynamoDB for file metadata.
* Build the share-link feature and ensure data consistency.

### Tasks during the week (13/07 - 17/07/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Create an on-demand DynamoDB table with fileId as the partition key; design keys around the access patterns (list by owner, look up by id). | 13/07/2026 | 13/07/2026 | [DynamoDB lab](https://000078.awsstudygroup.com) |
| Tue | Have Lambda write metadata with `put_item` on upload and read it back with `query`/`get_item` when listing files. | 14/07/2026 | 14/07/2026 |  |
| Wed | Handle S3-DynamoDB consistency: only write the DynamoDB item after the S3 upload succeeds, to avoid files without metadata. | 15/07/2026 | 15/07/2026 |  |
| Thu | Add scoped dynamodb:PutItem/GetItem/Query permissions for the table to the Lambda IAM Role. | 16/07/2026 | 16/07/2026 |  |
| Fri | Build the share-link feature (a presigned GET URL) and a view/download page; test end-to-end. | 17/07/2026 | 17/07/2026 |  |

### Results achieved

1. File metadata is stored and queried in DynamoDB, separate from file content on S3.
2. Handled consistency between S3 and DynamoDB.
3. Completed the share-link feature and tested the whole flow.
