---
title: "Sinh & test presigned URL"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### Presigned URL là gì

**Presigned URL** là đường dẫn có chữ ký trỏ tới một object trong S3, hiệu lực trong thời gian giới hạn, để trình duyệt tải lên/tải xuống trực tiếp trong khi bucket vẫn private.

#### Sinh presigned URL bằng boto3

Lambda tạo S3 client rồi sinh URL PUT để upload và URL GET để download, hết hạn sau 15 phút (`PRESIGN_EXPIRY = 900`).

```python
from botocore.config import Config

# Ép endpoint theo region + Signature V4 (xem ghi chú bên dưới).
s3 = boto3.client(
    "s3",
    region_name="ap-southeast-1",
    endpoint_url="https://s3.ap-southeast-1.amazonaws.com",
    config=Config(signature_version="s3v4", s3={"addressing_style": "virtual"}),
)

# URL upload (trình duyệt PUT file thẳng lên S3)
put_url = s3.generate_presigned_url(
    "put_object",
    Params={"Bucket": BUCKET, "Key": key, "ContentType": content_type},
    ExpiresIn=900,
)

# URL download / chia sẻ
get_url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": BUCKET, "Key": key},
    ExpiresIn=900,
)
```

{{% notice info %}}
**Ghi chú kỹ thuật.** S3 client mặc định dùng endpoint toàn cầu (`s3.amazonaws.com`), trả về **HTTP 307** khi upload lên bucket ở Singapore. Ép endpoint theo region kèm Signature V4 (như trên) để upload trả về HTTP 200.
{{% /notice %}}

#### Test upload qua presigned URL

Xin URL upload, rồi PUT file lên URL đó:

```bash
# 1) Xin API một presigned upload URL
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"test.txt","content_type":"text/plain"}'
# -> { "id": "...", "upload_url": "https://insightshare-files-khang-2352464.s3.ap-southeast-1...", "key": "..." }

# 2) Upload file trực tiếp lên S3 qua URL đó
curl -X PUT "<upload_url>" -H "Content-Type: text/plain" \
  --data-binary @test.txt -w "HTTP %{http_code}\n"
# -> HTTP 200
```

#### Kiểm tra object trong S3

Xác nhận file đã vào bucket:

```bash
aws s3 ls s3://insightshare-files-khang-2352464/ --recursive
# -> {file_id}/test.txt   (kích thước tính bằng byte)
```

#### Tóm tắt

Tầng lưu trữ dùng bucket private kèm CORS; file đi qua presigned URL mà bucket không bao giờ public. Cùng cơ chế này sinh URL GET để download và chia sẻ.
