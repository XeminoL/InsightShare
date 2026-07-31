---
title: "Worklog Tuần 7"
date: 2026-07-13
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7

* Tích hợp DynamoDB lưu metadata file.
* Phục vụ file qua link tải presigned và đảm bảo nhất quán dữ liệu.

### Các công việc trong tuần (13/07 - 17/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tạo bảng DynamoDB on-demand với partition key là `id`, thiết kế item theo cách truy cập. | 13/07/2026 | 13/07/2026 | [Serverless lab](https://000078.awsstudygroup.com) |
| 3 | Cho Lambda ghi metadata bằng `put_item` khi upload, đọc một item bằng `get_item`, liệt kê bằng `scan`. | 14/07/2026 | 14/07/2026 | |
| 4 | Xử lý nhất quán S3 và DynamoDB: ghi item metadata trước để mọi lượt upload đều được theo dõi, chỉ đánh dấu đã phân tích sau khi object nằm trên S3. | 15/07/2026 | 15/07/2026 | |
| 5 | Bổ sung quyền dynamodb PutItem/GetItem/UpdateItem/DeleteItem/Query/Scan giới hạn cho bảng vào IAM Role của Lambda. | 16/07/2026 | 16/07/2026 | |
| 6 | Cho `get_file` trả về presigned GET URL có thời hạn ngắn để frontend xem/tải file; kiểm thử end-to-end. | 17/07/2026 | 17/07/2026 | |

### Kết quả đạt được

1. Metadata file lưu và truy vấn từ DynamoDB, tách khỏi nội dung file trên S3.
2. Xử lý được nhất quán giữa S3 và DynamoDB.
3. File xem/tải được qua link presigned, kiểm thử cả luồng.