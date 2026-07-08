---
title: "Worklog Tuần 11"
date: 2026-08-10
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11

* Bổ sung tính năng nâng cao và hoàn thiện trải nghiệm.
* Học thêm các dịch vụ AWS liên quan để mở rộng kiến thức.

### Các công việc trong tuần (10/08 - 14/08/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Thêm tính năng xóa file (xóa object S3 và item DynamoDB cùng lúc), link tải presigned thời hạn ngắn, và kiểm tra dung lượng/loại file trước khi upload. | 10/08/2026 | 10/08/2026 |  |
| 3 | Cải thiện trang tìm kiếm: lọc theo nhãn Rekognition và hiển thị kết quả kèm ảnh thu nhỏ (presigned GET URL). | 11/08/2026 | 11/08/2026 |  |
| 4 | Tìm hiểu Amazon Cognito (user pool, hosted UI, JWT) để chuẩn bị thêm đăng nhập cho API. | 12/08/2026 | 12/08/2026 | [Cognito lab](https://000081.awsstudygroup.com) |
| 5 | Tìm hiểu Step Functions và cách dùng state machine điều phối pipeline nhiều bước Rekognition → Textract → Bedrock. | 13/08/2026 | 13/08/2026 | [Modernize](https://cloudjourney.awsstudygroup.com/4-modernize/) |
| 6 | Đo và tinh chỉnh hiệu năng: thời gian xử lý AI mỗi file và độ trễ tìm kiếm, đọc thời lượng từ CloudWatch. | 14/08/2026 | 14/08/2026 |  |

### Kết quả đạt được

1. Bổ sung các tính năng nâng cao (xóa file, link tải presigned thời hạn ngắn, giới hạn upload, và hỏi đáp tài liệu bằng Bedrock/Claude) làm phần đóng góp riêng.
2. Trang tìm kiếm dùng tốt hơn với bộ lọc nhãn và ảnh thu nhỏ.
3. Mở rộng kiến thức về Cognito và Step Functions, có hướng phát triển tiếp cho hệ thống.
