---
title: "Worklog Tuần 8"
date: 2026-07-20
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8

* Thêm lớp AI để hệ thống hiểu nội dung file.
* Dùng các dịch vụ AI gọi sẵn của AWS, không huấn luyện mô hình.

### Các công việc trong tuần (20/07 - 24/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Học workshop AI services của FCJ; chốt cách gọi Rekognition và Textract từ boto3 và vị trí kích hoạt trong luồng upload. | 20/07/2026 | 20/07/2026 | [AI services](https://000056.awsstudygroup.com) |
| 3 | Gọi Rekognition `detect_labels` (MaxLabels=10, MinConfidence=70) gắn nhãn ảnh khi upload; lưu nhãn để tìm kiếm. | 21/07/2026 | 21/07/2026 |  |
| 4 | Gọi Textract `detect_document_text` trích text từ PDF và ảnh chữ; file `.txt` đọc thẳng từ object trên S3. | 22/07/2026 | 22/07/2026 |  |
| 5 | Thêm endpoint `POST /files/{id}/ask`: gửi văn bản đã trích kèm câu hỏi tới một model Claude trên Amazon Bedrock (`invoke_model`) và trả về câu trả lời tiếng Việt; không có câu hỏi thì tóm tắt. | 23/07/2026 | 23/07/2026 | [Bedrock](https://docs.aws.amazon.com/bedrock/) |
| 6 | Mở rộng IAM Role với quyền rekognition/textract/bedrock; lưu nhãn và text vào DynamoDB; kiểm thử cả pipeline. | 24/07/2026 | 24/07/2026 |  |

### Kết quả đạt được

1. Ảnh upload được tự gắn nhãn bằng Rekognition; tài liệu được trích text bằng Textract (file `.txt` đọc trực tiếp).
2. Endpoint hỏi đáp tài liệu trả lời câu hỏi và tóm tắt bằng tiếng Việt qua Bedrock/Claude; xử lý mềm bằng thông báo HTTP 200 trong lúc inference quota Bedrock của tài khoản còn là 0.
3. Nhãn và text lưu vào DynamoDB, làm nền cho tìm kiếm; quyền AI theo least-privilege.
