---
title: "Viết Lambda function"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Mục tiêu

Một **Lambda function Python** cho back-end của InsightShare. Một function duy nhất điều hướng mọi route theo HTTP method và path: upload, list, search, analyze, ask, get, delete.

#### Tạo function

Tạo Lambda function trong region `ap-southeast-1`:

- **Runtime**: Python 3.13
- **Handler**: `lambda_function.handler`
- **Execution role**: `insightshare-lambda-role` (least-privilege, tạo ở phần 5.5)
- **Timeout**: 30s, **Memory**: 256 MB
- **Biến môi trường**: `BUCKET=insightshare-files-khang-2352464`, `TABLE=insightshare-files`

Bằng AWS CLI:

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

Màn Function overview cho thấy trigger API Gateway đã nối vào function:

![Đã tạo Lambda function](/images/5-Workshop/5.4-serverless-backend/lambda-function.png)

#### Handler

Handler đọc HTTP method và path từ sự kiện API Gateway (payload format v2) rồi điều hướng tới đúng hàm. `boto3` có sẵn trong runtime Lambda nên không cần đóng gói thêm.

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

Hàm upload sinh presigned URL (phần 5.3.2) và ghi dòng metadata ban đầu:

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

#### Deploy & test

Đóng gói file rồi test bằng một sự kiện API Gateway giả:

```bash
zip function.zip lambda_function.py
aws lambda invoke --function-name insightshare-api \
  --payload file://event.json --cli-binary-format raw-in-base64-out out.json
cat out.json
```

Lần invoke đầu trả về HTTP 200 kèm presigned URL thật và một dòng tương ứng trong DynamoDB, xác nhận cả quyền S3 lẫn DynamoDB đều hoạt động.
