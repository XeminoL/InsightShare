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
| 2 | Học workshop AI services của FCJ; chốt cách gọi Rekognition, Textract, Polly từ boto3 và vị trí kích hoạt trong luồng upload. | 20/07/2026 | 20/07/2026 | [AI services](https://000056.awsstudygroup.com) |
| 3 | Gọi Rekognition `detect_labels` (MaxLabels=10, MinConfidence=70) gắn nhãn ảnh và `detect_moderation_labels` để cờ nội dung nhạy cảm khi upload. | 21/07/2026 | 21/07/2026 |  |
| 4 | Gọi Textract `detect_document_text` trích text từ PDF và ảnh chữ, đọc thẳng file từ object trên S3. | 22/07/2026 | 22/07/2026 |  |
| 5 | Gọi Polly `synthesize_speech` (OutputFormat=mp3, chọn Voice) đọc phần text đã trích thành mp3 và lưu lên S3. | 23/07/2026 | 23/07/2026 |  |
| 6 | Mở rộng IAM Role với quyền rekognition/textract/polly; lưu nhãn và text vào DynamoDB; kiểm thử cả pipeline. | 24/07/2026 | 24/07/2026 |  |

### Kết quả đạt được

1. Ảnh upload được tự gắn nhãn và kiểm duyệt bằng Rekognition.
2. Tài liệu được trích text bằng Textract và đọc thành audio bằng Polly.
3. Nhãn và text lưu vào DynamoDB, làm nền cho tìm kiếm; quyền AI theo least-privilege và nằm trong Free Tier.
