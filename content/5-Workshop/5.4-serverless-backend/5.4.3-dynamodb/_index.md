---
title: "Integrate DynamoDB metadata"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

#### Goal

Store each file's **metadata** in **Amazon DynamoDB** so InsightShare can list, search and manage files. The AI labels and extracted text also live here, which is what makes content-based search possible.

#### Step 1: Create the DynamoDB table

Open the DynamoDB console (region `ap-southeast-1`) and choose **Create table**:

- **Table name**: `insightshare-files`
- **Partition key**: `id` (String), the same unique hex id used as the S3 key prefix, so one file maps to exactly one item and every read is a direct key lookup.
- No sort key, because each file is a single standalone item with no parent-child grouping.
- Capacity mode: **On-demand** (`PAY_PER_REQUEST`), which bills per request and needs no provisioned throughput, fitting the unpredictable, low-volume access of a demo.

![DynamoDB table](/images/5-Workshop/5.4-serverless-backend/dynamodb-table.png)

The screenshot confirms the table is Active with `id` as its partition key.

CLI equivalent:

```bash
aws dynamodb create-table --table-name insightshare-files \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

#### Step 2: Metadata attributes

Each item stores one file's information:

- `id` : primary key (a unique hex id)
- `filename`, `content_type`, `s3_key` : original name, MIME type, key in S3
- `labels` : list of AI labels (from Rekognition)
- `text` : extracted text (from Textract, or the file content for `.txt`)
- `search_blob` : lowercase labels + text, used for content search
- `size`, `uploaded_at` : object size (from the S3 `head_object`) and upload timestamp

#### Step 3: Wire Lambda to DynamoDB

The metadata row is written once on upload and updated once after analysis, so the table always reflects the file's current state. Lambda uses the `boto3` DynamoDB resource: `put_item` on upload (the empty row from 5.4.1), `update_item` after AI analysis to fill in labels and text, and `scan` for list/search:

```python
ddb = boto3.resource("dynamodb", region_name="ap-southeast-1")
table = ddb.Table("insightshare-files")

table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s",
    ExpressionAttributeNames={"#t": "text"},
    ExpressionAttributeValues={
        ":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower(),
    },
)
```

Note the `ExpressionAttributeNames={"#t": "text"}`: `text` is a reserved word in DynamoDB expressions, so writing `SET text=:t` directly is rejected. Aliasing it to the placeholder `#t` and mapping `#t` back to `text` lets the update run while still writing the attribute literally named `text`.

#### Step 4: Test

Call the upload API, then confirm a new item appears:

```bash
aws dynamodb scan --table-name insightshare-files --select COUNT
```

![Console: item in the DynamoDB table](/images/5-Workshop/5.4-serverless-backend/dynamodb-item.png)

The screenshot confirms the upload created one item keyed by its `id`, with the metadata attributes populated.

{{% notice info %}}
**Technical note.** The execution role initially granted `PutItem`/`GetItem`/`Query`/`Scan` but not `UpdateItem`, so `analyze` (which calls `update_item`) returned `AccessDeniedException ... not authorized to perform: dynamodb:UpdateItem`. Adding `dynamodb:UpdateItem` (and `s3:ListBucket`, needed elsewhere) to the role policy resolves it. The change did not take effect immediately: a warm Lambda execution environment caches the role credentials, so the new permission applied only after the function was updated and a fresh environment started. This is a caching effect, not a policy error, and is worth knowing when an IAM fix seems not to work.
{{% /notice %}}
