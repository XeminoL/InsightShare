---
title: "Worklog Tuần 7"
date: 2026-07-13
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7

* Tích hợp DynamoDB lưu metadata file.
* Xây tính năng chia sẻ link và đảm bảo nhất quán dữ liệu.

### Các công việc trong tuần (13/07 - 17/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tạo bảng DynamoDB on-demand với partition key là fileId, thiết kế khóa theo cách truy vấn (liệt kê theo owner, tra cứu theo id). | 13/07/2026 | 13/07/2026 | [DynamoDB lab](https://000078.awsstudygroup.com) |
| 3 | Cho Lambda ghi metadata bằng `put_item` khi upload và đọc lại bằng `query`/`get_item` khi liệt kê file. | 14/07/2026 | 14/07/2026 |  |
| 4 | Xử lý nhất quán S3 và DynamoDB: chỉ ghi item DynamoDB sau khi upload S3 thành công, tránh file đã lên nhưng thiếu metadata. | 15/07/2026 | 15/07/2026 |  |
| 5 | Bổ sung quyền dynamodb:PutItem/GetItem/Query giới hạn cho bảng vào IAM Role của Lambda. | 16/07/2026 | 16/07/2026 |  |
| 6 | Xây tính năng sinh link chia sẻ (presigned GET URL) và trang xem/tải file qua link; kiểm thử end-to-end. | 17/07/2026 | 17/07/2026 |  |

### Kết quả đạt được

1. Metadata file lưu và truy vấn từ DynamoDB, tách khỏi nội dung file trên S3.
2. Xử lý được nhất quán giữa S3 và DynamoDB.
3. Hoàn thiện tính năng chia sẻ link và kiểm thử cả luồng.
