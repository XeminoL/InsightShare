---
title: "Tích hợp DynamoDB metadata"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

#### Mục tiêu

Lưu **metadata** của mỗi file vào **Amazon DynamoDB** để InsightShare liệt kê, tìm kiếm và quản lý file. Nhãn AI và văn bản trích xuất cũng nằm ở đây, chính là thứ giúp tìm kiếm theo nội dung.

#### Bước 1: Tạo bảng DynamoDB

Mở DynamoDB console (region `ap-southeast-1`) và chọn **Create table**:

- **Table name**: `insightshare-files`
- **Partition key**: `id` (String), cùng mã hex duy nhất dùng làm tiền tố key S3, nên một file ứng đúng một item và mọi lần đọc là tra key trực tiếp.
- Không có sort key, vì mỗi file là một item độc lập, không có gom nhóm cha-con.
- Chế độ dung lượng: **On-demand** (`PAY_PER_REQUEST`), tính tiền theo request và không cần cấp phát throughput, hợp với mức truy cập thất thường, lượng thấp của bản demo.

![Bảng DynamoDB](/images/5-Workshop/5.4-serverless-backend/dynamodb-table.png)

Ảnh chụp xác nhận bảng ở trạng thái Active với partition key `id`.

Lệnh CLI tương đương:

```bash
aws dynamodb create-table --table-name insightshare-files \
  --attribute-definitions AttributeName=id,AttributeType=S \
  --key-schema AttributeName=id,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

#### Bước 2: Các thuộc tính metadata

Mỗi item lưu thông tin một file:

- `id` : khóa chính (mã hex duy nhất)
- `filename`, `content_type`, `s3_key` : tên gốc, kiểu MIME, key trong S3
- `labels` : danh sách nhãn AI (từ Rekognition)
- `text` : văn bản trích xuất (từ Textract, hoặc nội dung file với `.txt`)
- `search_blob` : nhãn + văn bản viết thường, dùng để tìm kiếm theo nội dung
- `size`, `uploaded_at` : kích thước object (từ `head_object` của S3) và thời điểm upload

#### Bước 3: Nối Lambda với DynamoDB

Dòng metadata được ghi một lần khi upload và cập nhật một lần sau khi phân tích, nên bảng luôn phản ánh trạng thái hiện tại của file. Lambda dùng DynamoDB resource của `boto3`: `put_item` khi upload (dòng rỗng từ 5.4.1), `update_item` sau khi phân tích AI để điền nhãn và văn bản, và `scan` cho list/search:

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

Lưu ý `ExpressionAttributeNames={"#t": "text"}`: `text` là từ khóa dành riêng trong expression của DynamoDB, nên viết thẳng `SET text=:t` sẽ bị từ chối. Đặt bí danh `#t` rồi ánh xạ `#t` về `text` cho phép update chạy mà vẫn ghi đúng thuộc tính tên `text`.

#### Bước 4: Test

Gọi API upload, rồi xác nhận item mới xuất hiện:

```bash
aws dynamodb scan --table-name insightshare-files --select COUNT
```

![Console: item trong bảng DynamoDB](/images/5-Workshop/5.4-serverless-backend/dynamodb-item.png)

Ảnh chụp xác nhận lần upload đã tạo một item khóa theo `id` của nó, với các thuộc tính metadata đã điền.

{{% notice info %}}
**Ghi chú kỹ thuật.** Execution role ban đầu có `PutItem`/`GetItem`/`Query`/`Scan` nhưng thiếu `UpdateItem`, nên gọi `analyze` (vốn gọi `update_item`) trả về `AccessDeniedException ... not authorized to perform: dynamodb:UpdateItem`. Thêm `dynamodb:UpdateItem` (và `s3:ListBucket`, cần ở nơi khác) vào policy của role để xử lý. Thay đổi không có hiệu lực ngay: môi trường chạy Lambda còn ấm giữ cache credentials của role, nên quyền mới chỉ áp dụng sau khi cập nhật lại function và một môi trường mới khởi tạo. Đây là hiệu ứng cache, không phải lỗi policy, và đáng biết khi một bản sửa IAM có vẻ chưa ăn.
{{% /notice %}}
