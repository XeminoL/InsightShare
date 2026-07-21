---
title: "Write the Lambda function"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Goal

One **Python Lambda function** for InsightShare's back-end. A single function dispatches all routes by HTTP method and path: upload, list, search, analyze, ask, get, delete.

#### Step 1: Create the function

Create a Lambda function in region `ap-southeast-1`:

- **Runtime**: Python 3.13
- **Handler**: `lambda_function.handler`
- **Execution role**: `insightshare-lambda-role` (least-privilege, created in section 5.5)
- **Timeout**: 30s, **Memory**: 256 MB
- **Environment variables**: `BUCKET=insightshare-files-khang-2352464`, `TABLE=insightshare-files`

With the AWS CLI:

```bash
aws lambda create-function \
  --function-name insightshare-api \
  --runtime python3.13 \
  --role arn:aws:iam::<account-id>:role/insightshare-lambda-role \
  --handler lambda_function.handler \
  --zip-file fileb://function.zip \
  --timeout 30 --memory-size 256 \
  --environment "Variables={BUCKET=insightshare-files-khang-2352464,TABLE=insightshare-files}"
```

The function overview shows the API Gateway trigger wired to the function:

![Lambda function created](/images/5-Workshop/5.4-serverless-backend/lambda-function.png)

#### Step 2: The handler

The handler reads the HTTP method and path from the API Gateway event (payload format v2) and routes to the right function. `boto3` ships with the Lambda runtime, so no extra packaging is needed.

```python
def handler(event, context):
    method = event["requestContext"]["http"]["method"]
    path = event.get("rawPath", "/")
    parts = [p for p in path.split("/") if p]

    if parts == ["files"] and method == "POST":
        return create_upload(event)
    if parts == ["files"] and method == "GET":
        return list_files(event)
    if parts == ["files", "search"] and method == "GET":
        return search_files(event)
    if len(parts) == 3 and parts[2] == "analyze" and method == "POST":
        return analyze(event, parts[1])
    if len(parts) == 3 and parts[2] == "ask" and method == "POST":
        return ask_document(event, parts[1])
    if len(parts) == 2 and parts[0] == "files" and method == "GET":
        return get_file(event, parts[1])
    if len(parts) == 2 and parts[0] == "files" and method == "DELETE":
        return delete_file(event, parts[1])
    return _resp(404, {"error": "route not found"})
```

The upload handler generates a presigned URL (section 5.3.2) and writes the initial metadata row:

```python
def create_upload(event):
    body = json.loads(event.get("body") or "{}")
    file_id = uuid.uuid4().hex
    key = f"{file_id}/{body['filename']}"
    put_url = s3.generate_presigned_url(
        "put_object",
        Params={"Bucket": BUCKET, "Key": key, "ContentType": body["content_type"]},
        ExpiresIn=900,
    )
    table.put_item(Item={
        "id": file_id, "filename": body["filename"],
        "content_type": body["content_type"], "s3_key": key,
        "labels": [], "text": "", "search_blob": body["filename"].lower(),
        "uploaded_at": int(time.time()),
    })
    return _resp(200, {"id": file_id, "upload_url": put_url, "key": key})
```

#### Step 3: Deploy & test

Package the single file and test it with a fake API Gateway event:

```bash
zip function.zip lambda_function.py
aws lambda invoke --function-name insightshare-api \
  --payload file://event.json --cli-binary-format raw-in-base64-out out.json
cat out.json
```

The first invoke returned HTTP 200 with a real presigned URL and a matching DynamoDB row, confirming both the S3 and DynamoDB permissions work.
