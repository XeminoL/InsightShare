---
title: "Worklog Tuần 9"
date: 2026-07-27
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9

* Thêm lớp AI để hệ thống hiểu nội dung file, theo từng user.
* Dùng các dịch vụ AI gọi sẵn của AWS, không huấn luyện mô hình.

### Các công việc trong tuần (27/07 - 31/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Học workshop AI services của FCJ; chốt cách gọi Rekognition và Textract từ boto3 và vị trí kích hoạt trong luồng upload. | 27/07/2026 | 27/07/2026 | [AI services](https://000056.awsstudygroup.com) |
| 3 | Gọi Rekognition `detect_labels` (MaxLabels=10, MinConfidence=70) gắn nhãn ảnh khi upload; lưu nhãn để tìm kiếm. | 28/07/2026 | 28/07/2026 |  |
| 4 | Gọi Textract `detect_document_text` trích text từ PDF và ảnh chữ; file `.txt` đọc thẳng từ object trên S3. | 29/07/2026 | 29/07/2026 |  |
| 5 | Thêm endpoint `POST /files/{id}/ask`: gửi văn bản đã trích kèm câu hỏi tới một model Claude trên Amazon Bedrock (`invoke_model`) và trả về câu trả lời theo ngôn ngữ câu hỏi; không có câu hỏi thì tóm tắt. | 30/07/2026 | 30/07/2026 | [Bedrock](https://docs.aws.amazon.com/bedrock/) |
| 6 | Mở rộng IAM Role với quyền rekognition/textract/bedrock theo least-privilege; lưu nhãn và text vào DynamoDB theo owner; kiểm thử cả pipeline. | 31/07/2026 | 31/07/2026 |  |

### Kết quả đạt được

1. Ảnh upload được tự gắn nhãn bằng Rekognition; tài liệu được trích text bằng Textract (file `.txt` đọc trực tiếp), cả hai lưu theo owner của file.
2. Endpoint hỏi đáp tài liệu trả lời câu hỏi và tóm tắt theo ngôn ngữ câu hỏi qua Bedrock/Claude. Textract và Bedrock được nối dây kèm xử lý fail-soft, nên một dịch vụ AI không sẵn sàng không làm hỏng luồng phân tích.
3. Nhãn và text lưu vào DynamoDB làm nền cho tìm kiếm sau này; quyền AI theo least-privilege.
