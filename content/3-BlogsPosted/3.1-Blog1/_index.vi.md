---
title: "Blog 1"
date: 2026-06-20
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---
# Hệ thống tóm tắt cuộc họp bằng AI với Amazon Bedrock và Amazon Transcribe

## Tóm tắt
Một hệ thống serverless của AWS biến file ghi âm cuộc họp thành bản tóm tắt có cấu trúc, hành động được, ghép nhiều dịch vụ AWS thành một ứng dụng generative AI với chi phí theo lượt dùng thấp.

## Nội dung chính

![Sơ đồ kiến trúc hệ thống tóm tắt cuộc họp](/images/3-Blog/blog1_architecture.png)

### Vấn đề
Nhiều tổ chức có nhiều dữ liệu âm thanh (cuộc họp, phỏng vấn) nhưng khó khai thác thông tin hữu ích: âm thanh là dữ liệu phi cấu trúc và nghe lại tốn thời gian.

### Giải pháp
Hệ thống xử lý theo luồng tự động:
1. **Amazon Transcribe** chuyển giọng nói trong file ghi âm thành văn bản.
2. **Amazon Bedrock** (mô hình Claude) đọc văn bản và sinh bản tóm tắt có cấu trúc: các bên liên quan, mục tiêu, hạng mục hành động và yêu cầu kỹ thuật.
3. **AWS Step Functions** điều phối toàn bộ luồng, **Lambda** xử lý từng bước, dữ liệu lưu ở **S3** và **DynamoDB**.
4. Giao diện là ứng dụng React, xác thực qua **Cognito**, truy vấn dữ liệu qua **AppSync** (GraphQL).

### Kết quả
- Bản tóm tắt gọn, có cấu trúc rõ ràng thay vì phải nghe lại cả cuộc họp.
- Kiến trúc serverless, tự động mở rộng theo tải.
- Bài gốc cho biết chi phí trung bình khoảng 0,98 USD mỗi cuộc họp.

### Bài học rút ra
Bài ghép nhiều dịch vụ AWS (Transcribe, Bedrock, Step Functions, Lambda, S3, DynamoDB) thành một ứng dụng hoàn chỉnh. Yêu cầu mô hình sinh bản tóm tắt theo cấu trúc định sẵn (bên liên quan, hành động, yêu cầu) cho kết quả dùng được trực tiếp thay vì một đoạn văn chung chung.

## Nguồn tham khảo
[Build an AI-powered automated summarization system with Amazon Bedrock and Amazon Transcribe](https://aws.amazon.com/blogs/machine-learning/build-an-ai-powered-automated-summarization-system-with-amazon-bedrock-and-amazon-transcribe-using-terraform/) (AWS Machine Learning Blog)

## Link bài đăng
https://www.facebook.com/share/p/1DGNJvxwZz/

## Hình ảnh
![Ảnh chụp bài đăng trên AWS Study Group](/images/3-Blog/blog1_post.png)
