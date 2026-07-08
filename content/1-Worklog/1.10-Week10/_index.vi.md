---
title: "Worklog Tuần 10"
date: 2026-08-03
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10

* Làm cho việc deploy và dọn dẹp lặp lại được, và kiểm thử end-to-end.
* Đo lường, tối ưu hiệu năng và ổn định hệ thống.

### Các công việc trong tuần (03/08 - 07/08/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Viết script cho bước deploy (đóng gói zip và chạy `aws lambda update-function-code`) để deploy lại sau khi đổi code chỉ bằng một lệnh. | 03/08/2026 | 03/08/2026 |  |
| 3 | Kiểm thử end-to-end và kiểm thử lỗi: file lớn, presigned link hết hạn và sai quyền. | 04/08/2026 | 04/08/2026 |  |
| 4 | Đo thời gian xử lý AI (Rekognition/Textract/Bedrock) và độ trễ tìm kiếm trên CloudWatch; xác định điểm nghẽn. | 05/08/2026 | 05/08/2026 |  |
| 5 | Tối ưu: tăng memory Lambda cho tác vụ AI, gọi AI bất đồng bộ qua S3 event để không chặn request upload. | 06/08/2026 | 06/08/2026 |  |
| 6 | Cải thiện xử lý lỗi và thông báo phản hồi (upload/tìm kiếm rõ hơn); chuẩn hóa mã lỗi trả về từ API. | 07/08/2026 | 07/08/2026 |  |

### Kết quả đạt được

1. Deploy và dọn dẹp đã được script hóa (zip + update-function-code; cleanup-aws.ps1), giảm thao tác tay và sai sót.
2. Hệ thống được kiểm thử qua các tình huống chính, kể cả các trường hợp lỗi (file lớn, link hết hạn, sai quyền).
3. Đo và tối ưu được hiệu năng (memory Lambda, xử lý AI bất đồng bộ), giảm độ trễ và ổn định hệ thống hơn.
