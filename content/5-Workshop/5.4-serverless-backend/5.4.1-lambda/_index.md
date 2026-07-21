---
title: "Write the Lambda function"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Goal

The Lambda is the only compute in InsightShare: API Gateway forwards every request to it, and it holds all business logic (presigning, metadata, AI calls) so no server is provisioned or patched. One **Python Lambda function** serves the whole back-end, dispatching all routes by HTTP method and path: upload, list, search, analyze, ask, get, delete. Keeping it in one function means one deploy artifact and one place for the shared S3/DynamoDB clients.

#### Step 1: Create the function

Create a Lambda function in region `ap-southeast-1`:

- **Runtime**: Python 3.13, matching the code and giving `boto3` in the runtime with no packaging.
- **Handler**: `lambda_function.handler`, the `handler` function in `lambda_function.py` that Lambda invokes per request.
- **Execution role**: `insightshare-lambda-role`, the least-privilege role the function assumes to reach S3, DynamoDB and the AI services.
- **Timeout**: 30s, high enough for a Textract or Bedrock call to finish; **Memory**: 256 MB, enough for JSON and a single file's text.
- **Environment variables**: `BUCKET=insightshare-files-khang-2352464` and `TABLE=insightshare-files`, so the bucket and table names stay out of the code.

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

Because API Gateway is set up with a single `$default` route, the handler itself does the routing. It reads the HTTP method and path from the API Gateway event (payload format v2, where the method is under `requestContext.http` and the path under `rawPath`) and dispatches to the matching function. `boto3` ships with the Lambda runtime, so no extra packaging is needed.

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

The upload handler starts the pipeline. It mints a unique `file_id`, builds the S3 key as `{file_id}/{filename}`, generates a presigned PUT URL, and writes an initial metadata row so the file is tracked before its bytes arrive. `labels`, `text` and `search_blob` start empty and are filled in later by `analyze`:

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

Because there are no third-party dependencies, deploying is zipping the one source file. The test sends a hand-written API Gateway event so the function runs without the API in front of it:

```bash
zip function.zip lambda_function.py
aws lambda invoke --function-name insightshare-api \
  --payload file://event.json --cli-binary-format raw-in-base64-out out.json
cat out.json
```

The first invoke returned HTTP 200 with a real presigned URL and a matching DynamoDB row, confirming both the S3 and DynamoDB permissions work.
