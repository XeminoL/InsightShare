---
title: "Worklog Tuần 11"
date: 2026-08-10
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11

* Kiểm thử end-to-end, tinh chỉnh hiệu năng và hoàn thiện hệ thống.
* Viết script dọn dẹp và xóa toàn bộ tài nguyên.

### Các công việc trong tuần (10/08 - 14/08/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Viết script deploy (đóng gói zip và chạy `aws lambda update-function-code`) để deploy lại sau khi đổi code chỉ bằng một lệnh. | 10/08/2026 | 10/08/2026 |  |
| 3 | Kiểm thử end-to-end và kiểm thử lỗi: file lớn, presigned link hết hạn, sai quyền, và tải đồng thời. | 11/08/2026 | 11/08/2026 | Postman |
| 4 | Đo thời gian xử lý AI và độ trễ tìm kiếm trên CloudWatch và tinh chỉnh memory Lambda để giảm thời lượng. | 12/08/2026 | 12/08/2026 |  |
| 5 | Viết script dọn dẹp (cleanup-aws.ps1) xóa toàn bộ stack; ghi chú Step Functions như cách điều phối pipeline AI nhiều bước về sau. | 13/08/2026 | 13/08/2026 | [Modernize](https://cloudjourney.awsstudygroup.com/4-modernize/) |
| 6 | Quay video demo và hoàn thiện báo cáo song ngữ. | 14/08/2026 | 14/08/2026 |  |

### Kết quả đạt được

1. Deploy và dọn dẹp đã được script hóa (zip + update-function-code; cleanup-aws.ps1), nên deploy lại và dọn tài nguyên chỉ còn một lệnh.
2. Hệ thống được kiểm thử end-to-end và qua các trường hợp lỗi; memory Lambda được tinh chỉnh từ thời lượng trên CloudWatch để giảm độ trễ.
3. Hệ thống hoàn thiện, có video demo và báo cáo song ngữ; Step Functions được ghi chú như hướng điều phối pipeline AI về sau.
