---
title: "Worklog Tuần 10"
date: 2026-08-03
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10

* Tìm kiếm file theo nội dung và đưa frontend lên S3 + CloudFront.
* Thêm giám sát CloudWatch và giảm chi phí.

### Các công việc trong tuần (03/08 - 07/08/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Xây tìm kiếm file theo nhãn Rekognition và text Textract trong DynamoDB, khớp từ khóa với thuộc tính nhãn và text đã lưu của owner. | 03/08/2026 | 03/08/2026 |  |
| 3 | Thêm endpoint `/ask` trả lời trên toàn thư viện và trích nguồn từng câu trả lời lấy từ file nào. | 04/08/2026 | 04/08/2026 |  |
| 4 | Host frontend lên S3 + CloudFront với HTTPS và Origin Access Control, để bucket file vẫn private phía sau distribution. | 05/08/2026 | 05/08/2026 | [CloudFront](https://cloudjourney.awsstudygroup.com/1-explore/) |
| 5 | Cho Lambda ghi log sang CloudWatch và tạo metric alarm trên metric Errors của hàm. | 06/08/2026 | 06/08/2026 |  |
| 6 | Tối ưu chi phí (lifecycle rule xóa object cũ trên S3, TTL cache CloudFront) và rà soát least-privilege toàn hệ thống. | 07/08/2026 | 07/08/2026 |  |

### Kết quả đạt được

1. Tìm được file theo nội dung (nhãn ảnh hoặc text tài liệu), và endpoint `/ask` trả lời trên toàn thư viện có trích nguồn file.
2. Frontend chạy trên S3 + CloudFront qua HTTPS với Origin Access Control; bucket file vẫn private.
3. Log Lambda và alarm Errors đã có trên CloudWatch; chi phí được giảm bằng lifecycle S3 và TTL CloudFront, quyền được rà soát theo least-privilege.
