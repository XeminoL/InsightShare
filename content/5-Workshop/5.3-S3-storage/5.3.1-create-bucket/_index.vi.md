---
title: "Tạo S3 bucket & cấu hình"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Bước 1: Tạo S3 bucket

Mở **S3 console** (region `ap-southeast-1`) và chọn **Create bucket**:

- **Bucket name**: `insightshare-files-khang-2352464` (tên S3 là duy nhất toàn cầu, nên thêm hậu tố cá nhân để tránh trùng).
- **Region**: Asia Pacific (Singapore) `ap-southeast-1`.

Sau khi tạo, bucket mở ra ở trạng thái trống:

![Đã tạo S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-bucket-created.png)

Kiểm tra region bằng CLI:

```bash
aws s3api get-bucket-location --bucket insightshare-files-khang-2352464
```

#### Bước 2: Tắt public access

Giữ **tick cả 4 ô Block Public Access** để bucket luôn ở chế độ private:

![Console: S3 Block Public Access đang bật](/images/5-Workshop/5.3-S3-storage/s3-block-public-access.png)

```bash
aws s3api get-public-access-block --bucket insightshare-files-khang-2352464
```

#### Bước 3: Bật versioning

Bật **Bucket Versioning** để giữ các phiên bản cũ của object:

![Console: S3 versioning đã bật](/images/5-Workshop/5.3-S3-storage/s3-versioning.png)

```bash
aws s3api get-bucket-versioning --bucket insightshare-files-khang-2352464
```

#### Bước 4: Cấu hình CORS

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
