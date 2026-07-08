---
title: "Back-end serverless: Lambda + API Gateway + DynamoDB"
date: 2026-07-29
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Tổng quan

Phần này gồm phần lõi của InsightShare: back-end serverless và frontend.

+ **Lambda (Python)** xử lý nghiệp vụ: nhận yêu cầu, sinh presigned URL, ghi metadata.
+ **API Gateway** expose các API để frontend gọi.
+ **DynamoDB** lưu metadata của từng file (tên, thời gian, kích thước, link chia sẻ, nhãn AI, text trích...).
+ **Lớp AI** (Rekognition, Textract, Polly) phân tích nội dung file để gắn nhãn, kiểm duyệt, trích text và đọc audio.
+ **Frontend** (web tĩnh) được host trên **S3 + CloudFront**.

Sau khi hoàn thành phần này, toàn bộ luồng từ trình duyệt → API Gateway → Lambda → S3/DynamoDB → lớp AI sẽ hoạt động end-to-end.

#### Nội dung

- [Viết Lambda function](5.4.1-lambda/)
- [Tạo API Gateway](5.4.2-api-gateway/)
- [Tích hợp DynamoDB metadata](5.4.3-dynamodb/)
- [Frontend S3 + CloudFront](5.4.4-frontend-cloudfront/)
- [Thêm AI: Rekognition, Textract, Polly](5.4.5-ai-services/)
