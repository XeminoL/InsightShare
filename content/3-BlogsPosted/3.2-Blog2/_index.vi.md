---
title: "Blog 2"
date: 2026-07-11
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---
# Xây ứng dụng nhiều bước với AWS Lambda Durable Functions

## Tóm tắt
Lambda Durable Functions của AWS cho phép xây các ứng dụng nhiều bước và luồng AI ngay trong Lambda, với việc quản lý trạng thái và phục hồi lỗi do runtime lo thay vì một kho trạng thái riêng.

## Nội dung chính

![Sơ đồ Lambda Durable Functions](/images/3-Blog/blog2_architecture.png)

### Vấn đề
Làm một luồng xử lý dài hạn (chuỗi phê duyệt, pipeline AI nhiều bước) buộc lập trình viên tự viết nhiều thứ để lưu trạng thái, xử lý lỗi và nối các dịch vụ lại, tốn công và dễ sinh lỗi.

### Giải pháp
Durable Functions đưa khả năng đó vào ngay trong Lambda với hai thành phần chính:
1. **Steps:** mỗi bước được tự động lưu tiến trình (checkpoint) và tự retry khi lỗi, theo cơ chế "checkpoint và replay".
2. **Waits:** cho phép tạm dừng thực thi tới một năm mà không tính phí compute trong lúc chờ, rất hợp để chờ phê duyệt của con người hoặc phản hồi từ API bên ngoài.

Ngoài ra tính năng có idempotency tích hợp (gọi trùng tên execution sẽ trả kết quả đã có, tránh chạy lại), và tích hợp EventBridge để phát trạng thái thực thi.

### Điểm đáng chú ý
- Không cần tự dựng hạ tầng lưu trạng thái hay xử lý retry.
- Tạm dừng dài hạn không tốn phí compute.
- Hỗ trợ Node.js và Python phiên bản mới; triển khai được bằng AWS SAM.

### Bài học rút ra
Với các luồng nghiệp vụ dài hoặc AI workflow cần chờ, Durable Functions tự lo trạng thái và retry thay cho một kho trạng thái riêng, và bước chờ phê duyệt của con người không tốn compute nhàn rỗi.

## Nguồn tham khảo
[Build multi-step applications and AI workflows with AWS Lambda durable functions](https://aws.amazon.com/blogs/aws/build-multi-step-applications-and-ai-workflows-with-aws-lambda-durable-functions/) (AWS News Blog)

## Link bài đăng
https://www.facebook.com/share/p/1D53oYGQmP/

## Hình ảnh
![Ảnh chụp bài đăng trên AWS Study Group](/images/3-Blog/blog2_post.png)
