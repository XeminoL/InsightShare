---
title: "Tạo API Gateway"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### Mục tiêu

Tạo một **API Gateway (HTTP API)** làm cổng công khai để frontend gọi tới Lambda. HTTP API rẻ và đơn giản hơn REST API, đủ dùng cho dự án này.

#### Bước 1: Tạo API

API Gateway là cửa HTTPS công khai: nó kết thúc TLS, áp CORS, và (khi thêm authorizer ở 5.4.5) kiểm tra token Cognito trước khi bất kỳ request nào tới Lambda. Vì Lambda đã tự điều hướng theo method và path bên trong, chỉ cần một route **`$default`** chuyển mọi thứ tới Lambda là đủ, nên không phải khai báo route hai lần. `--cors-configuration` ở đây cho trình duyệt gọi API cross-origin, tương ứng với CORS của S3 ở 5.3:

```bash
aws apigatewayv2 create-api \
  --name insightshare-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:ap-southeast-1:<account-id>:function:insightshare-api \
  --cors-configuration "AllowOrigins=*,AllowMethods=GET,POST,DELETE,OPTIONS,AllowHeaders=*"
```

Dùng `--target` sẽ tự tạo integration Lambda, route `$default` và stage `$default` (auto-deploy). API Gateway vẫn cần một resource-based permission rõ ràng trên function trước khi được gọi; `--source-arn` giới hạn quyền đó về đúng API này nên không API nào khác gọi được function:

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

Ảnh chụp xác nhận đúng một route `$default` trỏ vào integration Lambda.

#### Bước 2: Các route do Lambda xử lý

Đây là các endpoint logic mà frontend dùng; tất cả đều đi vào qua route `$default` duy nhất và được tách bên trong Lambda. Gộp lại chúng tạo thành pipeline: upload một file, phân tích nó, rồi liệt kê, tìm kiếm hoặc hỏi đáp trên kết quả.

| Method | Path | Mục đích |
|---|---|---|
| POST | `/files` | Xin presigned upload URL + tạo metadata |
| POST | `/files/{id}/analyze` | Chạy Rekognition/Textract trên object đã upload |
| POST | `/files/{id}/ask` | Hỏi đáp về tài liệu (Bedrock/Claude, tiếng Việt) |
| POST | `/ask` | Hỏi đáp trên toàn thư viện (Bedrock/Claude, kèm tệp nguồn) |
| GET | `/files` | Liệt kê tất cả file |
| GET | `/files/search?q=` | Tìm kiếm theo nội dung (nhãn + văn bản trích xuất) |
| GET | `/files/{id}` | Metadata của một file + presigned download URL |
| DELETE | `/files/{id}` | Xóa object + metadata |

#### Bước 3: Test API

```bash
API="https://<api-id>.execute-api.ap-southeast-1.amazonaws.com"

curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"hello.txt","content_type":"text/plain"}'

curl "$API/files"
```

{{% notice info %}}
**Ghi chú kỹ thuật.** DynamoDB resource của `boto3` trả mọi số đã lưu (như `uploaded_at` và `size`) dưới dạng `Decimal` của Python để giữ độ chính xác. `json.dumps` không có quy tắc cho `Decimal`, nên lần `GET /files` đầu tiên lỗi `Object of type Decimal is not JSON serializable`. Một JSON encoder tùy chỉnh chuyển `Decimal` sang `int` (hoặc `float` khi có phần thập phân) và được truyền vào `json.dumps` trong mọi response, nên API luôn trả số JSON hợp lệ.

```python
class _DecimalEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(o, decimal.Decimal):
            return int(o) if o % 1 == 0 else float(o)
        return super().default(o)
```
{{% /notice %}}

Sau khi sửa, `GET /files` và `GET /files/search?q=` đều trả JSON sạch.
