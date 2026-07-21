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
+ **API Gateway** expose các API để frontend gọi, kèm một JWT authorizer cho token Cognito.
+ **Amazon Cognito** đăng nhập người dùng qua Hosted UI, để mỗi người chỉ thấy file của mình.
+ **DynamoDB** lưu metadata của từng file (tên, thời gian, kích thước, nhãn AI, text trích...).
+ **Lớp AI** (Rekognition, Textract, Bedrock/Claude) gắn nhãn ảnh, trích văn bản tài liệu, và trả lời câu hỏi về một tài liệu.
+ **Frontend** (web tĩnh) được host trên **S3 + CloudFront**.

Sau khi hoàn thành phần này, toàn bộ luồng từ trình duyệt → API Gateway → Lambda → S3/DynamoDB → lớp AI sẽ hoạt động end-to-end.

#### Nội dung

- [Viết Lambda function](5.4.1-lambda/)
- [Tạo API Gateway](5.4.2-api-gateway/)
- [Tích hợp DynamoDB metadata](5.4.3-dynamodb/)
- [Frontend S3 + CloudFront](5.4.4-frontend-cloudfront/)
- [Đăng nhập theo người dùng với Amazon Cognito](5.4.5-cognito-auth/)
- [Thêm AI: Rekognition, Textract, Bedrock (Claude)](5.4.6-ai-services/)
