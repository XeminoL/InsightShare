---
title: "Lưu trữ file với S3 + presigned URL"
date: 2026-07-29
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Tổng quan

Tầng lưu trữ giữ file người dùng trong một **Amazon S3 bucket private** (không công khai). Việc tải lên/tải xuống đi qua **presigned URL** do back-end sinh ra, nên trình duyệt truyền file **trực tiếp** lên S3 mà không đi vòng qua Lambda, đồng thời bucket vẫn không public.

Các bước chính của phần này:

+ Tạo S3 bucket lưu file (region `ap-southeast-1`).
+ Tắt public access và bật mã hóa.
+ Cấu hình **CORS** để trình duyệt gọi được tới bucket.
+ Sinh **presigned URL** bằng **boto3** và kiểm tra upload/download.

#### Nội dung

- [Tạo S3 bucket & cấu hình](5.3.1-create-bucket/)
- [Sinh & test presigned URL](5.3.2-presigned-url/)
