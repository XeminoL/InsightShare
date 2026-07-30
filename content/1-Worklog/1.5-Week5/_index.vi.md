---
title: "Worklog Tuần 5"
date: 2026-06-29
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5

* Đưa back-end lên AWS Lambda, chạy hoàn toàn serverless.
* Gắn IAM Role và kiểm thử Lambda độc lập.

### Các công việc trong tuần (29/06 - 03/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Viết một Lambda handler thuần điều hướng event API Gateway HTTP API theo method và path, đóng gói thành file zip để deploy cho runtime Python. | 29/06/2026 | 29/06/2026 | [Serverless lab](https://000078.awsstudygroup.com) |
| 3 | Tinh chỉnh hàm: đặt bộ nhớ 256 MB và timeout 30s để một lời gọi AI đủ thời gian xong, đọc cấu hình từ biến môi trường. | 30/06/2026 | 30/06/2026 |  |
| 4 | Tạo execution IAM Role cho Lambda với AWSLambdaBasicExecutionRole cùng một inline policy least-privilege chỉ trỏ tới bucket cần thiết. | 01/07/2026 | 01/07/2026 | [IAM Role](https://000048.awsstudygroup.com) |
| 5 | Tách biến môi trường cho local và cloud (tên bucket, region, tên bảng) để cùng một code chạy được ở cả hai. | 02/07/2026 | 02/07/2026 |  |
| 6 | Kiểm thử Lambda bằng `invoke` trực tiếp với event payload mẫu và đọc log CloudWatch trước khi nối API. | 03/07/2026 | 03/07/2026 |  |

### Kết quả đạt được

1. Back-end chạy hoàn toàn serverless trên Lambda, bộ nhớ và timeout đã đặt vừa cho các lời gọi AI.
2. Lambda có IAM Role riêng theo least-privilege thay vì cấp quyền rộng.
3. Hàm Lambda kiểm thử độc lập đạt yêu cầu trước khi tích hợp.
