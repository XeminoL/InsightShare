---
title: "Tự đánh giá"
date: 2026-06-01
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Kỳ thực tập tại **Công ty TNHH Amazon Web Services Việt Nam** (chương trình First Cloud AI Journey) diễn ra từ **01/06/2026** đến **15/08/2026**.

Sản phẩm chính là **InsightShare**, ứng dụng web upload và chia sẻ ảnh/tài liệu theo hướng serverless trên AWS. Quá trình làm việc trực tiếp với S3, Lambda, API Gateway, DynamoDB, CloudFront, IAM, CloudWatch; ngôn ngữ Python (boto3); tài liệu kỹ thuật viết song ngữ.

Bảng dưới tự đánh giá theo tám tiêu chí, mỗi tiêu chí một mức: Tốt / Khá.

| STT | Tiêu chí | Tốt | Khá | Nhận xét |
| --- | --- | --- | --- | --- |
| 1 | **Kiến thức và kỹ năng chuyên môn** | x | | Áp dụng được các dịch vụ AWS cốt lõi vào một dự án end-to-end. |
| 2 | **Khả năng học hỏi** | x | | Tự học các dịch vụ mới (Lambda, presigned URL, CloudFront) qua tài liệu và lab. |
| 3 | **Tính chủ động** | x | | Chủ động trong thiết kế và tự tìm hiểu các dịch vụ cần dùng. |
| 4 | **Kỷ luật** | | x | Giữ lịch lên văn phòng cố định và nộp worklog hàng tuần đúng hạn. |
| 5 | **Giao tiếp** | x | | Báo cáo công việc rõ bằng văn bản và tài liệu kỹ thuật song ngữ. |
| 6 | **Hợp tác nhóm** | | x | Hòa nhập với nhóm, hỏi mentor khi vướng, và hỗ trợ khi cần. |
| 7 | **Tư duy giải quyết vấn đề** | x | | Chẩn đoán các lỗi tích hợp (CORS, IAM, cold start, S3 307, `Decimal` của DynamoDB) từ log CloudWatch. |
| 8 | **Đóng góp cho dự án** | x | | Tự làm InsightShare end-to-end và bổ sung tính năng thêm (xóa file, link có hạn, giới hạn upload). |

### Bài học kỹ thuật

* Hành vi presigned URL khác nhau giữa các region S3: endpoint toàn cầu mặc định trả HTTP 307 với bucket ở Singapore, nên client phải ép endpoint theo region và dùng Signature V4 để có URL chạy được.
* Lambda đang chạy cache credentials của môi trường thực thi, nên thay đổi policy IAM chỉ có hiệu lực sau khi cập nhật lại hoặc deploy lại function để tạo môi trường chạy mới.
* `text` là từ khóa dành riêng của DynamoDB, nên update expression chạm vào nó cần `ExpressionAttributeNames` để đặt bí danh.
* DynamoDB trả số dưới dạng `Decimal` của Python, mà `json.dumps` không serialize được; mọi response API cần một JSON encoder tùy chỉnh.
* Các lệnh gọi AI (Textract, Bedrock) nên fail soft và được bọc lại, để một dịch vụ không sẵn sàng suy giảm êm thay vì làm hỏng cả luồng upload và phân tích.
* Log CloudWatch là công cụ chính để chẩn đoán lỗi runtime serverless, vì không có máy chủ nào để gắn vào.
