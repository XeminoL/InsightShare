---
title: "Tích hợp DynamoDB metadata"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

#### Mục tiêu

Lưu **metadata** của mỗi file vào **Amazon DynamoDB** để InsightShare liệt kê, tìm kiếm và quản lý file. Nhãn AI và văn bản trích xuất cũng nằm ở đây, chính là thứ giúp tìm kiếm theo nội dung.

#### Tạo bảng DynamoDB

Mở DynamoDB console (region `ap-southeast-1`) và chọn **Create table**:

- **Table name**: `insightshare-files`
- **Partition key**: `id` (String)
- Không có sort key
- Chế độ dung lượng: **On-demand** (`PAY_PER_REQUEST`), không phải tinh chỉnh capacity

![Bảng DynamoDB](/images/5-Workshop/5.4-serverless-backend/dynamodb-table.png)

Lệnh CLI tương đương:

```bash
aws dynamodb create-table --table-name insightshare-files \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

#### Các thuộc tính metadata

Mỗi item lưu thông tin một file:

- `id` : khóa chính (mã hex duy nhất)
- `filename`, `content_type`, `s3_key` : tên gốc, kiểu MIME, key trong S3
- `labels` : danh sách nhãn AI (từ Rekognition)
- `text` : văn bản trích xuất (từ Textract, hoặc nội dung file với `.txt`)
- `search_blob` : nhãn + văn bản viết thường, dùng để tìm kiếm theo nội dung
- `size`, `uploaded_at` : kích thước object (từ `head_object` của S3) và thời điểm upload

#### Nối Lambda với DynamoDB

Lambda dùng DynamoDB resource của `boto3`. `put_item` khi upload, `update_item` sau khi phân tích AI, và `scan` cho list/search:

```python
ddb = boto3.resource("dynamodb", region_name="ap-southeast-1")
table = ddb.Table("insightshare-files")

# sau khi phân tích AI, lưu nhãn + văn bản và dựng lại search blob
table.update_item(
    Key={"id": file_id},
    UpdateExpression="SET labels=:l, #t=:t, search_blob=:s",
    ExpressionAttributeNames={"#t": "text"},   # 'text' là từ khóa dành riêng
    ExpressionAttributeValues={
        ":l": labels, ":t": text,
        ":s": (" ".join(labels) + " " + text).lower(),
    },
)
```

Lưu ý `ExpressionAttributeNames={"#t": "text"}`: `text` là từ khóa dành riêng của DynamoDB, nên phải đặt bí danh trong update expression.

#### Test

Gọi API upload, rồi xác nhận item mới xuất hiện:

```bash
aws dynamodb scan --table-name insightshare-files --select COUNT
# -> "Count": 1
```

{{% notice info %}}
**Ghi chú kỹ thuật.** Execution role có `PutItem`/`GetItem`/`Query`/`Scan` nhưng thiếu `UpdateItem`, nên gọi `analyze` trả về `AccessDeniedException ... not authorized to perform: dynamodb:UpdateItem`. Thêm `dynamodb:UpdateItem` (và `s3:ListBucket`) vào policy của role để xử lý. Lambda đang chạy giữ credentials cache, nên thay đổi policy IAM chỉ có hiệu lực sau khi cập nhật lại function để tạo môi trường chạy mới.
{{% /notice %}}
