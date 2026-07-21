---
title: "Viết Lambda function"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

#### Mục tiêu

Lambda là phần compute duy nhất của InsightShare: API Gateway chuyển mọi request tới nó, và nó giữ toàn bộ nghiệp vụ (presign, metadata, gọi AI) nên không phải cấp phát hay vá server nào. Một **Lambda function Python** phục vụ cả back-end, điều hướng mọi route theo HTTP method và path: upload, list, search, analyze, ask, get, delete. Gộp trong một function nghĩa là một artifact deploy và một chỗ dùng chung các client S3/DynamoDB.

#### Bước 1: Tạo function

Tạo Lambda function trong region `ap-southeast-1`:

- **Runtime**: Python 3.13, khớp với code và cho sẵn `boto3` trong runtime, không phải đóng gói.
- **Handler**: `lambda_function.handler`, hàm `handler` trong `lambda_function.py` mà Lambda gọi cho mỗi request.
- **Execution role**: `insightshare-lambda-role`, role least-privilege mà function assume để chạm S3, DynamoDB và các dịch vụ AI (tạo ở phần 5.5).
- **Timeout**: 30s, đủ cho một lời gọi Textract hoặc Bedrock hoàn tất; **Memory**: 256 MB, đủ cho JSON và văn bản của một file.
- **Biến môi trường**: `BUCKET=insightshare-files-khang-2352464` và `TABLE=insightshare-files`, để tên bucket và bảng nằm ngoài code.

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

#### Bước 2: Handler

Vì API Gateway được cấu hình với một route `$default` duy nhất (phần 5.4.2), chính handler làm việc điều hướng. Nó đọc HTTP method và path từ sự kiện API Gateway (payload format v2, method nằm dưới `requestContext.http` và path dưới `rawPath`) rồi dispatch tới đúng hàm. `boto3` có sẵn trong runtime Lambda nên không cần đóng gói thêm.

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

Hàm upload mở đầu pipeline. Nó sinh một `file_id` duy nhất, dựng key S3 dạng `{file_id}/{filename}`, sinh presigned PUT URL (phần 5.3.2), và ghi dòng metadata ban đầu để file được theo dõi trước khi bytes tới. `labels`, `text` và `search_blob` khởi tạo rỗng và được `analyze` điền về sau:

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

#### Bước 3: Deploy & test

Vì không có phụ thuộc bên thứ ba, deploy chỉ là zip một file nguồn. Bài test gửi một sự kiện API Gateway viết tay để function chạy mà không cần API đứng trước:

```bash
zip function.zip lambda_function.py
aws lambda invoke --function-name insightshare-api \
  --payload file://event.json --cli-binary-format raw-in-base64-out out.json
cat out.json
```

Lần invoke đầu trả về HTTP 200 kèm presigned URL thật và một dòng tương ứng trong DynamoDB, xác nhận cả quyền S3 lẫn DynamoDB đều hoạt động.
