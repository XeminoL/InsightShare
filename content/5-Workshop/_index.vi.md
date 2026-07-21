---
title: "Workshop"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
alwaysopen: false
---

# InsightShare: Xây dựng nền tảng chia sẻ ảnh & tài liệu tích hợp AI trên AWS (Serverless)

#### Tổng quan

**InsightShare** là một ứng dụng web để tải lên, phân tích và chia sẻ ảnh và tài liệu, được xây dựng hoàn toàn theo kiến trúc **serverless** trên AWS. Khác với lưu trữ thông thường, ngay khi một file được tải lên, hệ thống dùng các dịch vụ AI của AWS để hiểu nội dung của nó, nhờ vậy người dùng có thể tìm file theo nội dung bên trong chứ không chỉ theo tên file.

Phạm vi triển khai:
- Lưu file trong **Amazon S3**, chia sẻ qua **presigned URL** có thời hạn
- Back-end serverless trên **AWS Lambda** (Python) sau **Amazon API Gateway**
- Metadata, nhãn AI và văn bản trích xuất trong **Amazon DynamoDB** để tìm kiếm theo nội dung
- Lớp hiểu nội dung dùng **Amazon Rekognition**, **Amazon Textract** và **Amazon Bedrock** (Claude)
- Frontend tĩnh phân phối qua **Amazon CloudFront** (HTTPS), giám sát bằng **Amazon CloudWatch**
- Quyền truy cập tối thiểu (least-privilege) bằng **AWS IAM** trên mọi dịch vụ

Dự án dùng kiến trúc serverless gồm:
- **Lưu trữ & chia sẻ**: Amazon S3 lưu file riêng tư với presigned URL; metadata trong Amazon DynamoDB
- **Back-end serverless**: AWS Lambda (Python) sau Amazon API Gateway xử lý toàn bộ nghiệp vụ
- **Hiểu nội dung bằng AI**: Amazon Rekognition (gắn nhãn ảnh), Amazon Textract (trích văn bản), Amazon Bedrock/Claude (hỏi đáp và tóm tắt tài liệu bằng tiếng Việt), đều gọi sẵn được, không cần huấn luyện mô hình
- **Tìm kiếm thông minh**: nhãn và văn bản trích xuất được lưu trong DynamoDB để tìm file theo nội dung
- **Phân phối & giám sát**: Amazon CloudFront phân phối HTTPS, Amazon CloudWatch ghi log và số liệu
- **Bảo mật**: IAM Role theo nguyên tắc tối thiểu quyền

#### Nội dung

1. [Tổng quan](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Lưu trữ file với S3 + presigned URL](5.3-S3-storage/)
4. [Back-end serverless: Lambda + API Gateway + DynamoDB](5.4-serverless-backend/)
5. [Giám sát & Bảo mật (CloudWatch + IAM)](5.5-Policy/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)
