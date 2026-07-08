---
title: "Tạo API Gateway"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### Mục tiêu

Tạo một **API Gateway (HTTP API)** làm cổng công khai để frontend gọi tới Lambda. HTTP API rẻ và đơn giản hơn REST API, đủ dùng cho dự án này.

#### Tạo API

Vì Lambda đã tự điều hướng theo method và path bên trong, chỉ cần một route **`$default`** chuyển mọi thứ tới Lambda là đủ:

```bash
aws apigatewayv2 create-api \
  --name insightshare-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:ap-southeast-1:<account-id>:function:insightshare-api \
  --cors-configuration "AllowOrigins=*,AllowMethods=GET,POST,DELETE,OPTIONS,AllowHeaders=*"
# -> trả về ApiId và ApiEndpoint (invoke URL)
```

Dùng `--target` sẽ tự tạo integration Lambda, route `$default` và stage `$default` (auto-deploy). Sau đó cấp quyền cho API Gateway gọi Lambda:

```bash
aws lambda add-permission \
  --function-name insightshare-api \
  --statement-id apigw-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:ap-southeast-1:<account-id>:<api-id>/*/*"
```

Màn Routes cho thấy route `$default` duy nhất do API Gateway quản lý, đã tích hợp với Lambda:

![Route của API Gateway](/images/5-Workshop/5.4-serverless-backend/apigateway-routes.png)

#### Các route do Lambda xử lý

| Method | Path | Mục đích |
|---|---|---|
| POST | `/files` | Xin presigned upload URL + tạo metadata |
| POST | `/files/{id}/analyze` | Chạy Rekognition/Textract trên object đã upload |
| POST | `/files/{id}/ask` | Hỏi đáp về tài liệu (Bedrock/Claude, tiếng Việt) |
| POST | `/ask` | Ask across the whole library (Bedrock/Claude, with source files) |
| GET | `/files` | Liệt kê tất cả file |
| GET | `/files/search?q=` | Tìm kiếm theo nội dung (nhãn + văn bản trích xuất) |
| GET | `/files/{id}` | Metadata của một file + presigned download URL |
| DELETE | `/files/{id}` | Xóa object + metadata |

#### Test API

```bash
API="https://<api-id>.execute-api.ap-southeast-1.amazonaws.com"

# POST /files -> presigned URL
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"hello.txt","content_type":"text/plain"}'

# GET /files -> danh sách
curl "$API/files"
```

{{% notice info %}}
**Ghi chú kỹ thuật.** DynamoDB trả các số (như `uploaded_at`) dưới dạng `Decimal` của Python, mà `json.dumps` không serialize được, sinh lỗi `Object of type Decimal is not JSON serializable`. Một JSON encoder tùy chỉnh chuyển `Decimal` sang `int`/`float` và được dùng cho mọi response.

```python
class _DecimalEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(o, decimal.Decimal):
            return int(o) if o % 1 == 0 else float(o)
        return super().default(o)
# ... json.dumps(body, cls=_DecimalEncoder)
```
{{% /notice %}}

Sau khi sửa, `GET /files` và `GET /files/search?q=` đều trả JSON sạch.
