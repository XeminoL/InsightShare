---
title: "Blog 3"
date: 2026-08-01
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---
# AWS Lambda MicroVMs: sandbox cách ly, giữ trạng thái để chạy code không tin cậy

## Tóm tắt
AWS Lambda MicroVMs, một compute primitive serverless công bố tháng 6/2026, chạy code do người dùng gửi lên hoặc do AI sinh ra trong môi trường cách ly, giữ trạng thái: mức cách ly ngang máy ảo, khởi động gần như tức thì từ snapshot và giữ trạng thái tới 8 giờ, mà không phải quản lý máy chủ.

## Nội dung chính

![Sơ đồ kiến trúc AWS Lambda MicroVMs](/images/3-Blog/blog3_architecture.png)

### Vấn đề
Các ứng dụng như trợ lý lập trình AI, trình thông dịch code tương tác hay công cụ quét lỗ hổng đều cần chạy đoạn code chúng không tin cậy. Các lựa chọn cũ đều phải đánh đổi: máy ảo đầy đủ cách ly tốt nhưng mất vài phút để khởi động; container khởi động nhanh nhưng cần gia cố nhiều mới an toàn cho code lạ; Lambda function thông thường khởi động nhanh nhưng sinh ra cho việc ngắn, phi trạng thái, theo sự kiện, không hợp phiên dài cần giữ trạng thái. Lambda MicroVMs lấp khoảng trống này.

### Cách hoạt động
1. Một Dockerfile cùng code được đóng gói thành file zip trên Amazon S3. Lambda chạy Dockerfile, khởi động ứng dụng, rồi chụp một snapshot Firecracker của bộ nhớ và ổ đĩa đang chạy. Đây là MicroVM Image.
2. Một phiên bắt đầu từ ARN của image và một chính sách nhàn rỗi. Lambda cấp một ID riêng và trả về một endpoint HTTPS riêng, không cần cấu hình mạng. MicroVM phục hồi từ snapshot thay vì khởi động nguội, nên khởi động và phục hồi gần như tức thì.
3. Mỗi MicroVM giữ nguyên bộ nhớ, ổ đĩa và tiến trình đang chạy qua các phiên. Khi nhàn rỗi, nó được tạm dừng tự động để tiết kiệm chi phí, và khôi phục toàn bộ trạng thái ở request kế tiếp.

### Điểm đáng chú ý
- Cách ly ở mức máy ảo, dùng Firecracker, chính công nghệ đứng sau hàng nghìn tỷ lượt gọi Lambda mỗi tháng. Không dùng chung kernel giữa các phiên.
- Trạng thái được giữ tới 8 giờ mỗi phiên. Gói đã cài, mô hình đã nạp và file đang làm việc vẫn còn sau khi tạm dừng rồi phục hồi.
- Mỗi MicroVM dùng tới 16 vCPU, 32 GB bộ nhớ và 32 GB ổ đĩa (ARM64). Ra mắt ở năm khu vực, trong đó có Asia Pacific (Tokyo).
- Nó bổ trợ cho Lambda function chứ không thay thế: một function có thể gọi MicroVM khi cần chạy code cách ly và giữ trạng thái.

### Bài học rút ra
MicroVMs lấp khoảng trống giữa Lambda và EC2: mức cách ly ngang máy ảo nhưng không phải chờ khởi động vài phút và không phải quản lý máy chủ. Với nhu cầu chạy an toàn code do AI sinh ra hoặc do người dùng gửi lên, việc gộp cách ly, phục hồi nhanh và giữ trạng thái vào một primitive quản lý sẵn là hướng đáng dùng.

## Nguồn tham khảo
[Run isolated sandboxes with full lifecycle control: AWS Lambda introduces MicroVMs](https://aws.amazon.com/blogs/aws/run-isolated-sandboxes-with-full-lifecycle-control-aws-lambda-introduces-microvms/) (AWS News Blog)

## Link bài đăng
https://www.facebook.com/share/p/18fFbiYK9m/

## Hình ảnh
![Ảnh chụp bài đăng trên AWS Study Group](/images/3-Blog/blog3_post.png)
