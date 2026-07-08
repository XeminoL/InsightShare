---
title: "Worklog Tuần 2"
date: 2026-06-08
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2

* Thiết kế kiến trúc InsightShare và viết Proposal.
* Thiết lập tài khoản AWS an toàn để bắt đầu làm.
* Tìm hiểu presigned URL và các dịch vụ AI sẽ dùng.

### Các công việc trong tuần (08/06 - 12/06/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Học IAM: user, group, role, phân biệt identity policy và resource policy, và nguyên tắc least-privilege để sau áp riêng cho từng dịch vụ. | 08/06/2026 | 08/06/2026 | [AWS IAM](https://000002.awsstudygroup.com) |
| 3 | Vẽ sơ đồ kiến trúc serverless của InsightShare bằng draw.io: trình duyệt → CloudFront/S3 → API Gateway → Lambda → S3/DynamoDB và nhánh Rekognition/Textract/Polly. | 09/06/2026 | 09/06/2026 | [draw.io](https://draw.io/) |
| 4 | Viết Proposal: vấn đề, giải pháp, bảng dịch vụ AWS (10 dịch vụ) và lý do chọn từng dịch vụ, đều ở region ap-southeast-1. | 10/06/2026 | 10/06/2026 |  |
| 5 | Tạo tài khoản AWS, bật MFA cho root, tạo IAM user riêng để không dùng root, cấu hình AWS CLI bằng `aws configure` và đặt cảnh báo Budget theo tháng. | 11/06/2026 | 11/06/2026 |  |
| 6 | Thiết kế bảng DynamoDB cho metadata (partition key fileId, các thuộc tính owner, thời gian, nhãn AI và text trích); tìm hiểu presigned URL và các API AI detect_labels, detect_document_text, synthesize_speech. | 12/06/2026 | 12/06/2026 | [AI services](https://000056.awsstudygroup.com) |

### Kết quả đạt được

1. Có sơ đồ kiến trúc và Proposal hoàn chỉnh, giải thích được lựa chọn từng dịch vụ.
2. Thiết lập tài khoản AWS theo chuẩn bảo mật: MFA, IAM user riêng, Budgets, CLI.
3. Hiểu cơ chế presigned URL và nắm sơ bộ cách gọi các dịch vụ AI.
