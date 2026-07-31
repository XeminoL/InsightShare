---
title: "Workshop"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
alwaysopen: false
---

# InsightShare: Xây dựng nền tảng chia sẻ ảnh & tài liệu tích hợp AI trên AWS

## Tổng quan

**InsightShare** là một ứng dụng web để tải lên, phân tích và chia sẻ ảnh và tài liệu, được xây dựng hoàn toàn theo kiến trúc **serverless** trên AWS. Sau khi một file được tải lên, hệ thống gọi các dịch vụ AI của AWS để đọc nội dung của nó rồi lưu lại kết quả.

Phạm vi triển khai:
- **Amazon S3** lưu file riêng tư, chia sẻ qua **presigned URL** có thời hạn
- Back-end serverless trên **AWS Lambda** sau **Amazon API Gateway**
- Metadata, nhãn AI và văn bản trích xuất trong **Amazon DynamoDB**, làm nền cho tìm kiếm theo nội dung
- Lớp hiểu nội dung với **Amazon Rekognition**, **Amazon Textract** và **Amazon Bedrock**
- Frontend tĩnh phân phối qua **Amazon CloudFront**, giám sát bằng **Amazon CloudWatch**
- Quyền truy cập tối thiểu bằng **AWS IAM** trên mọi dịch vụ

## Nội dung

1. [Tổng quan](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Lưu trữ file với S3 + presigned URL](5.3-S3-storage/)
4. [Back-end serverless: Lambda + API Gateway + DynamoDB](5.4-serverless-backend/)
5. [Giám sát & Bảo mật](5.5-Policy/)
6. [Dọn dẹp tài nguyên](5.6-Cleanup/)