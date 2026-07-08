---
title: "Worklog Tuần 4"
date: 2026-06-22
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4

* Đưa lưu trữ file lên Amazon S3 với presigned URL.
* Áp dụng IAM least-privilege và cấu hình CORS cho tầng lưu trữ.

### Các công việc trong tuần (22/06 - 26/06/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tạo S3 bucket private ở ap-southeast-1, bật Block Public Access và mã hóa mặc định SSE-S3. | 22/06/2026 | 22/06/2026 | [Amazon S3](https://000057.awsstudygroup.com) |
| 3 | Viết hàm sinh presigned URL cho upload/download bằng boto3 (`generate_presigned_url`, method put_object/get_object, hết hạn 900s); test bằng curl. | 23/06/2026 | 23/06/2026 |  |
| 4 | Cấu hình rule CORS của bucket (allowed origins, method PUT/GET, headers) để trình duyệt upload trực tiếp qua presigned URL. | 24/06/2026 | 24/06/2026 |  |
| 5 | Thay lớp lưu trữ local bằng S3 sau interface đã tách ở tuần 3, không phải sửa logic nghiệp vụ. | 25/06/2026 | 25/06/2026 |  |
| 6 | Viết IAM policy least-privilege giới hạn s3:PutObject/GetObject chỉ trên ARN của bucket này; kiểm thử upload/download và các tình huống lỗi (URL hết hạn, sai key). | 26/06/2026 | 26/06/2026 |  |

### Kết quả đạt được

1. File lưu an toàn trên S3, upload/download qua presigned URL mà bucket không public.
2. Thay được nền lưu trữ sang S3 gần như không sửa code nghiệp vụ nhờ đã tách interface.
3. Áp dụng least-privilege từ sớm và kiểm thử được các tình huống lỗi.
