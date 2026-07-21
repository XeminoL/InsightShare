---
title: "Tạo S3 bucket & cấu hình"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Bước 1: Tạo S3 bucket

Bucket này là nơi duy nhất lưu mọi file được upload; về sau Lambda đọc/ghi object ở đây và sinh presigned URL trỏ vào nó. Mở **S3 console** (region `ap-southeast-1`) và chọn **Create bucket**:

- **Bucket name**: `insightshare-files-khang-2352464` (tên S3 là duy nhất toàn cầu, nên thêm hậu tố cá nhân để tránh trùng).
- **Region**: Asia Pacific (Singapore) `ap-southeast-1`, cùng region với Lambda để request nằm trong region và presign dùng endpoint theo region.

Sau khi tạo, bucket mở ra ở trạng thái trống:

![Đã tạo S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-bucket-created.png)

Ảnh chụp xác nhận bucket tồn tại ở `ap-southeast-1` và chưa có object nào. Kiểm tra region bằng CLI:

```bash
aws s3api get-bucket-location --bucket insightshare-files-khang-2352464
```

#### Bước 2: Tắt public access

Bucket chứa file người dùng, tuyệt đối không được để cả thế giới đọc; quyền truy cập được cấp theo từng object theo từng request qua presigned URL, nên bản thân bucket không cần public. Giữ **tick cả 4 ô Block Public Access**: bốn thiết lập này chặn ACL public, bỏ qua ACL public đang có, chặn bucket policy public, và giới hạn policy public liên tài khoản, nên không tổ hợp ACL hay policy nào có thể vô tình lộ object.

![Console: S3 Block Public Access đang bật](/images/5-Workshop/5.3-S3-storage/s3-block-public-access.png)

Ảnh chụp xác nhận cả 4 ô đang bật. CLI cho ra cùng trạng thái:

```bash
aws s3api get-public-access-block --bucket insightshare-files-khang-2352464
```

#### Bước 3: Bật versioning

Versioning giữ lại bản cũ của object khi bị ghi đè hoặc xóa, nên một lần upload trùng key hay xóa nhầm vẫn khôi phục được thay vì mất file. Bật **Bucket Versioning**:

![Console: S3 versioning đã bật](/images/5-Workshop/5.3-S3-storage/s3-versioning.png)

Ảnh chụp xác nhận versioning đang ở trạng thái Enabled. CLI cho kết quả tương tự:

```bash
aws s3api get-bucket-versioning --bucket insightshare-files-khang-2352464
```

#### Bước 4: Cấu hình CORS

Trình duyệt tải file trực tiếp lên và tải xuống từ S3 qua presigned URL, và các request đó phát từ origin của trang web chứ không phải từ domain của S3, nên S3 từ chối trừ khi CORS cho phép rõ origin đó. `AllowedMethods` liệt kê `PUT` cho upload và `GET` cho download, hai verb duy nhất presigned URL dùng ở đây; `ExposeHeaders` trả về `ETag` để trình duyệt đọc được hash của object vừa upload; `MaxAgeSeconds` cache preflight CORS trong 3000 giây để tránh thêm một vòng request mỗi lần. Áp cấu hình CORS sau (mức demo cho phép mọi origin; khi lên production nên giới hạn `AllowedOrigins` về đúng domain web):

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

Khi bucket đã private, có versioning và bật CORS, bước tiếp theo sinh các presigned URL để trình duyệt chạm tới nó mà không phải mở bucket.
