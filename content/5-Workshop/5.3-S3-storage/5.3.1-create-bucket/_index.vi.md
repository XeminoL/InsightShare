---
title: "Tạo S3 bucket & cấu hình"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Tạo S3 bucket

Mở **S3 console** (region `ap-southeast-1`) và chọn **Create bucket**:

- **Bucket name**: `insightshare-files-khang-2352464` (tên S3 là duy nhất toàn cầu, nên thêm hậu tố cá nhân để tránh trùng).
- **Region**: Asia Pacific (Singapore) `ap-southeast-1`.
- **Block Public Access**: giữ **tick cả 4 ô** (bucket phải ở chế độ private).
- **Bucket Versioning**: **Enable** (giữ các phiên bản cũ của object).

Sau khi tạo, bucket mở ra ở trạng thái trống:

![Đã tạo S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-bucket-created.png)

Có thể kiểm tra lại cấu hình bằng CLI:

```bash
aws s3api get-bucket-location --bucket insightshare-files-khang-2352464
# -> "LocationConstraint": "ap-southeast-1"

aws s3api get-public-access-block --bucket insightshare-files-khang-2352464
# -> cả bốn giá trị đều true (BlockPublicAcls, IgnorePublicAcls,
#    BlockPublicPolicy, RestrictPublicBuckets)

aws s3api get-bucket-versioning --bucket insightshare-files-khang-2352464
# -> "Status": "Enabled"
```

#### Cấu hình CORS

Trình duyệt tải file trực tiếp lên và tải xuống từ S3 qua presigned URL. Để làm được điều đó, bucket phải cho phép yêu cầu cross-origin. Áp cấu hình CORS sau (mức demo cho phép mọi origin; khi lên production nên giới hạn `AllowedOrigins` về đúng domain web):

```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT"],
      "AllowedOrigins": ["*"],
      "ExposeHeaders": ["ETag"],
      "MaxAgeSeconds": 3000
    }
  ]
}
```

Áp và kiểm tra bằng CLI:

```bash
aws s3api put-bucket-cors --bucket insightshare-files-khang-2352464 \
  --cors-configuration file://cors.json

aws s3api get-bucket-cors --bucket insightshare-files-khang-2352464
```

#### Ghi chú

- Object được lưu theo tiền tố `{file_id}/{filename}`, nên mỗi file nằm trong thư mục riêng đặt theo id duy nhất.
- Lambda đọc tên bucket `insightshare-files-khang-2352464` từ biến môi trường `BUCKET`.
