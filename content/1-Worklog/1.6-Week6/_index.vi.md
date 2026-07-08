---
title: "Worklog Tuần 6"
date: 2026-07-06
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6

* Công khai API cho frontend qua API Gateway.
* Hoàn thiện luồng end-to-end trên cloud.

### Các công việc trong tuần (06/07 - 10/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tạo HTTP API Gateway tích hợp Lambda proxy, định nghĩa route cho từng endpoint (POST /files, GET /files, GET /files/{id}). | 06/07/2026 | 06/07/2026 | [API Gateway lab](https://000079.awsstudygroup.com) |
| 3 | Cấu hình CORS ở tầng API Gateway (allowed origins, method, headers) và thêm quyền cho gateway gọi Lambda. | 07/07/2026 | 07/07/2026 |  |
| 4 | Chuyển frontend sang gọi invoke URL của API Gateway thay cho localhost. | 08/07/2026 | 08/07/2026 |  |
| 5 | Kiểm thử API bằng Postman và từ frontend; xử lý lỗi CORS preflight, quyền IAM và mapping của proxy. | 09/07/2026 | 09/07/2026 | Postman |
| 6 | Rà soát và hoàn thiện luồng đầy đủ trên cloud: frontend → API Gateway → Lambda → presigned URL → upload thẳng lên S3. | 10/07/2026 | 10/07/2026 |  |

### Kết quả đạt được

1. API Gateway cung cấp endpoint công khai, frontend gọi được với CORS đúng.
2. Frontend đã chuyển hẳn sang API Gateway thay vì back-end local.
3. Tự gỡ được các lỗi tích hợp thường gặp và hoàn thiện luồng end-to-end trên cloud.
