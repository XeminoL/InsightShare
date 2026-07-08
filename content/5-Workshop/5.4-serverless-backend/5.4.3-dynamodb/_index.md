---
title: "Integrate DynamoDB metadata"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

#### Goal

Store each file's **metadata** in **Amazon DynamoDB** so InsightShare can list, search and manage files. The AI labels and extracted text also live here, which is what makes content-based search possible.

#### Create the DynamoDB table

Open the DynamoDB console (region `ap-southeast-1`) and choose **Create table**:

- **Table name**: `insightshare-files`
- **Partition key**: `id` (String)
- No sort key
- Capacity mode: **On-demand** (`PAY_PER_REQUEST`), no capacity to tune

![DynamoDB table](/images/5-Workshop/5.4-serverless-backend/dynamodb-table.png)

CLI equivalent:

```bash
aws dynamodb create-table --table-name insightshare-files \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

#### Metadata attributes

Each item stores one file's information:

- `id` : primary key (a unique hex id)
- `filename`, `content_type`, `s3_key` : original name, MIME type, key in S3
- `labels` : list of AI labels (from Rekognition)
- `text` : extracted text (from Textract, or the file content for `.txt`)
- `search_blob` : lowercase labels + text, used for content search
- `share_token`, `uploaded_at` : share link token and timestamp

#### Wire Lambda to DynamoDB

Lambda uses the `boto3` DynamoDB resource. `put_item` on upload, `update_item` after AI analysis, and `scan` for list/search:

```python
ddb = boto3.resource("dynamodb", region_name="ap-southeast-1")
table = ddb.Table("insightshare-files")

# after AI analysis, store labels + text and rebuild the search blob
table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s",
    ExpressionAttributeNames={"#t": "text"},   # 'text' is a reserved word
    ExpressionAttributeValues={
        ":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower(),
    },
)
```

Note the `ExpressionAttributeNames={"#t": "text"}`: `text` is a DynamoDB reserved word, so it must be aliased in the update expression.

#### Test

Call the upload API, then confirm a new item appears:

```bash
aws dynamodb scan --table-name insightshare-files --select COUNT
# -> "Count": 1
```

{{% notice info %}}
**Technical note.** The execution role granted `PutItem`/`GetItem`/`Query`/`Scan` but not `UpdateItem`, so `analyze` returned `AccessDeniedException ... not authorized to perform: dynamodb:UpdateItem`. Adding `dynamodb:UpdateItem` (and `s3:ListBucket`) to the role's policy resolves it. A running Lambda caches its credentials, so an IAM policy change takes effect only after the function is updated to spin up a fresh execution environment.
{{% /notice %}}
