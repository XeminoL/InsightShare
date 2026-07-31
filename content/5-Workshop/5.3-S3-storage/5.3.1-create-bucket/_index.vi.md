---
title: "Tạo S3 bucket & cấu hình"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Bước 1: Tạo S3 bucket

Bucket này là nơi duy nhất lưu mọi file được upload; về sau Lambda đọc/ghi object ở đây và sinh presigned URL trỏ vào nó. Mở **S3 console** và chọn **Create bucket**:

- **Bucket name**: `insightshare-files-khang-2352464`.
- **Region**: Asia Pacific `ap-southeast-1`, cùng region với Lambda, nên request không đi ra ngoài region và presigned URL ký theo endpoint của region đó.

Sau khi tạo, bucket mở ra ở trạng thái trống:

![Đã tạo S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-bucket-created.png)

Kiểm tra region bằng CLI:

```bash
aws s3api get-bucket-location --bucket insightshare-files-khang-2352464
```

#### Bước 2: Tắt public access

Bucket chứa file người dùng, tuyệt đối không được để cả thế giới đọc; quyền truy cập được cấp theo từng object theo từng request qua presigned URL, nên bản thân bucket không cần public. Giữ **tick cả 4 ô Block Public Access**: bốn thiết lập này chặn ACL public, bỏ qua ACL public đang có, chặn bucket policy public, và giới hạn policy public liên tài khoản, nên không tổ hợp ACL hay policy nào có thể vô tình lộ object.

![Console: S3 Block Public Access đang bật](/images/5-Workshop/5.3-S3-storage/s3-block-public-access.png)

Kiểm tra lại trạng thái bằng CLI:

```bash
aws s3api get-public-access-block --bucket insightshare-files-khang-2352464
```

#### Bước 3: Bật versioning

Versioning giữ lại bản cũ của object khi bị ghi đè hoặc xóa, nên một lần upload trùng key hay xóa nhầm vẫn khôi phục được thay vì mất file. Bật **Bucket Versioning**:

![Console: S3 versioning đã bật](/images/5-Workshop/5.3-S3-storage/s3-versioning.png)

Kiểm tra bằng CLI:

```bash
aws s3api get-bucket-versioning --bucket insightshare-files-khang-2352464
```

#### Bước 4: Cấu hình CORS

Trình duyệt truyền file trực tiếp với S3 qua presigned URL. Các request đó phát từ origin của trang web, không phải từ domain của S3, nên S3 từ chối nếu CORS không cho phép origin đó.

`AllowedMethods` chỉ liệt kê `PUT` và `GET`, hai verb mà presigned URL dùng. `ExposeHeaders` trả `ETag` để trình duyệt đọc được hash của object vừa upload. `MaxAgeSeconds` cache preflight 3000 giây, nên request sau trình duyệt khỏi hỏi lại.

Áp cấu hình sau:

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

- Object được lưu theo tiền tố `{file_id}/{filename}`, nên mỗi file nằm trong thư mục riêng đặt theo id duy nhất, cũng chính là key của item DynamoDB nối object với metadata của nó.
- Lambda đọc tên bucket `insightshare-files-khang-2352464` từ biến môi trường `BUCKET`, nên code không hard-code tên bucket.