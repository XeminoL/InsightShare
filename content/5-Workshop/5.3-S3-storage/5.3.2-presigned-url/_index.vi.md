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

Presigned URL mang sẵn chữ ký của Lambda role trong query string, nên trình duyệt gọi thẳng S3 cho đúng một object và một verb mà không cần bất kỳ credential AWS nào của mình. Hạn ngắn 15 phút (`PRESIGN_EXPIRY = 900`) giới hạn thời gian một link bị lộ còn dùng được. Client ép endpoint theo region và Signature V4 vì lý do ở ghi chú kỹ thuật bên dưới.

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
**Ghi chú kỹ thuật.** S3 client mặc định ký theo endpoint toàn cầu (`s3.amazonaws.com`). Bucket ở `ap-southeast-1` đáp lại endpoint đó bằng **HTTP 307 Temporary Redirect** trỏ về host theo region, và trình duyệt bỏ các header đã ký khi redirect, nên lệnh `PUT` thất bại. Đặt `endpoint_url` thành `https://s3.ap-southeast-1.amazonaws.com` để S3 trả lời thẳng không redirect, và `signature_version="s3v4"` ký bằng SigV4 mà endpoint theo region yêu cầu. Kết hợp cả hai làm upload trả về HTTP 200 ngay lần request đầu.
{{% /notice %}}

#### Bước 2: Test upload qua presigned URL

Đây chính là luồng upload hai lời gọi mà trình duyệt thực hiện: `POST /files` xin Lambda một URL đã ký, rồi một lệnh `PUT` gửi bytes thẳng lên S3. Cờ `-w "HTTP %{http_code}"` in ra mã trạng thái để thấy được 200.

```bash
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"test.txt","content_type":"text/plain"}'

curl -X PUT "<upload_url>" -H "Content-Type: text/plain" \
  --data-binary @test.txt -w "HTTP %{http_code}\n"
```

#### Bước 3: Kiểm tra object trong S3

Object xuất hiện dưới tiền tố `{file_id}/{filename}` của nó, chứng minh lệnh PUT presigned đã vào được bucket private mà bucket không cần public. Chính object vừa upload này là thứ lớp AI đọc về sau để sinh nhãn và văn bản.

```bash
aws s3 ls s3://insightshare-files-khang-2352464/ --recursive
```

![Console: object đã upload trong S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-object-uploaded.png)

