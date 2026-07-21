---
title: "Sinh & test presigned URL"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### Presigned URL là gì

**Presigned URL** là đường dẫn có chữ ký trỏ tới một object trong S3, hiệu lực trong thời gian giới hạn, để trình duyệt tải lên/tải xuống trực tiếp trong khi bucket vẫn private.

#### Bước 1: Sinh presigned URL bằng boto3

Lambda tạo S3 client rồi sinh URL PUT để upload và URL GET để download, hết hạn sau 15 phút (`PRESIGN_EXPIRY = 900`).

```python
from botocore.config import Config

s3 = boto3.client(
    "s3",
    region_name="ap-southeast-1",
    endpoint_url="https://s3.ap-southeast-1.amazonaws.com",
    config=Config(signature_version="s3v4", s3={"addressing_style": "virtual"}),
)

put_url = s3.generate_presigned_url(
    "put_object",
    Params={"Bucket": BUCKET, "Key": key, "ContentType": content_type},
    ExpiresIn=900,
)

get_url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": BUCKET, "Key": key},
    ExpiresIn=900,
)
```

{{% notice info %}}
**Ghi chú kỹ thuật.** S3 client mặc định dùng endpoint toàn cầu (`s3.amazonaws.com`), trả về **HTTP 307** khi upload lên bucket ở Singapore. Ép endpoint theo region kèm Signature V4 (như trên) để upload trả về HTTP 200.
{{% /notice %}}

#### Bước 2: Test upload qua presigned URL

Xin URL upload, rồi PUT file lên URL đó:

```bash
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"test.txt","content_type":"text/plain"}'

curl -X PUT "<upload_url>" -H "Content-Type: text/plain" \
  --data-binary @test.txt -w "HTTP %{http_code}\n"
```

#### Bước 3: Kiểm tra object trong S3

Xác nhận file đã vào bucket:

```bash
aws s3 ls s3://insightshare-files-khang-2352464/ --recursive
```

![Console: object đã upload trong S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-object-uploaded.png)

