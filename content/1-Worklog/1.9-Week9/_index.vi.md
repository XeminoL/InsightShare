---
title: "Worklog Tuần 9"
date: 2026-07-27
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9

* Xây tìm kiếm thông minh theo nhãn và nội dung.
* Đưa frontend lên CloudFront, thêm giám sát và tối ưu.

### Các công việc trong tuần (27/07 - 31/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Xây tìm kiếm file theo nhãn Rekognition và text Textract trong DynamoDB, khớp từ khóa với thuộc tính nhãn và text đã lưu. | 27/07/2026 | 27/07/2026 |  |
| 3 | Host frontend lên S3 + CloudFront với HTTPS và Origin Access Control, để bucket vẫn private phía sau distribution. | 28/07/2026 | 28/07/2026 | [CloudFront](https://cloudjourney.awsstudygroup.com/1-explore/) |
| 4 | Cho Lambda ghi log sang CloudWatch và tạo metric alarm trên metric Errors của hàm. | 29/07/2026 | 29/07/2026 |  |
| 5 | Tối ưu chi phí (lifecycle rule xóa object cũ trên S3, TTL cache CloudFront) và rà soát bảo mật cùng least-privilege toàn hệ thống. | 30/07/2026 | 30/07/2026 |  |
| 6 | Đưa hạ tầng vào code bằng AWS SAM (template.yaml) để deploy hoặc xóa stack bằng một lệnh. | 31/07/2026 | 31/07/2026 | [AWS SAM](https://000080.awsstudygroup.com) |

### Kết quả đạt được

1. Tìm được file theo nội dung (nhãn ảnh hoặc text tài liệu), không chỉ theo tên.
2. Frontend chạy qua CloudFront có HTTPS; hệ thống có log và alarm trên CloudWatch.
3. Giảm chi phí, siết bảo mật và quản lý toàn bộ hạ tầng bằng SAM.
