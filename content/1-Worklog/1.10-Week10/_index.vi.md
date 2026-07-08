---
title: "Worklog Tuần 10"
date: 2026-08-03
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10

* Tự động hóa triển khai bằng CI/CD và kiểm thử end-to-end.
* Đo lường, tối ưu hiệu năng và ổn định hệ thống.

### Các công việc trong tuần (03/08 - 07/08/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Thiết lập CI/CD bằng GitHub Actions chạy `sam build` và `sam deploy` khi push lên nhánh main, xác thực vào AWS qua repository secrets. | 03/08/2026 | 03/08/2026 | [CI/CD lab](https://000084.awsstudygroup.com) |
| 3 | Kiểm thử end-to-end và kiểm thử lỗi: file lớn, presigned link hết hạn, sai quyền và tải đồng thời. | 04/08/2026 | 04/08/2026 |  |
| 4 | Đo thời gian xử lý AI (Rekognition/Textract/Polly) và độ trễ tìm kiếm trên CloudWatch; xác định điểm nghẽn. | 05/08/2026 | 05/08/2026 |  |
| 5 | Tối ưu: tăng memory Lambda cho tác vụ AI, gọi AI bất đồng bộ qua S3 event để không chặn request upload. | 06/08/2026 | 06/08/2026 |  |
| 6 | Cải thiện xử lý lỗi và thông báo phản hồi (upload/tìm kiếm rõ hơn); chuẩn hóa mã lỗi trả về từ API. | 07/08/2026 | 07/08/2026 |  |

### Kết quả đạt được

1. Có pipeline CI/CD tự build và deploy khi push, giảm thao tác thủ công và rủi ro khi triển khai.
2. Hệ thống được kiểm thử kỹ qua nhiều tình huống, kể cả tải đồng thời và các trường hợp lỗi.
3. Đo và tối ưu được hiệu năng (memory Lambda, xử lý AI bất đồng bộ), giảm độ trễ và ổn định hệ thống hơn.
