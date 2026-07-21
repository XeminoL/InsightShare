---
title: "Sự kiện 1"
date: 2026-06-06
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

| Trường | Chi tiết |
| --- | --- |
| Tên sự kiện | FCAJ Community Day - June 2026 |
| Thời gian | [DD/MM/2026, 09:00-12:00] |
| Địa điểm | Tầng 26 và 36, tòa nhà Bitexco Financial Tower, số 2 Hải Triều, phường Sài Gòn, TP. Hồ Chí Minh |
| Vai trò | Người tham dự |

### Nội dung chính
Sự kiện cộng đồng hàng tháng, các diễn giả từ nhiều công ty chia sẻ kinh nghiệm thực tế khi xây dựng AI trên AWS. Năm phần trình bày:

- **Sự nghiệp trong kỷ nguyên AI (Cloud Thinker).** Một founder kể lại hành trình từ vận hành server on-prem đến Solution Architect ở AWS, rồi khởi nghiệp. Thông điệp cho sinh viên: thị trường tuyển developer và các vai trò cloud đang thu hẹp khi doanh nghiệp áp dụng AI, nên tích lũy kinh nghiệm thực tế sớm và học cách làm việc tốt với công cụ AI. Anh giới thiệu một nền tảng agentic hỗ trợ kỹ sư trong điều tra sự cố, review code trước khi lên production, tối ưu chi phí (FinOps) và kiểm thử bảo mật, và giải thích vì sao họ chọn một single agent thiết kế tốt thay vì multi-agent cho phần lớn tác vụ.
- **Voice AI (Renova Cloud, RhoAI).** So sánh hai kiến trúc: speech-to-speech trực tiếp, và speech-to-text rồi LLM rồi text-to-speech. Kiến trúc ba bước được ưu tiên cho tiếng Việt vì tiếng Việt là ngôn ngữ ít tài nguyên, và vì đi qua text cho phép kiểm soát nội dung model nói ra và dùng tool calling ổn định. Xử lý lượt nói, ngắt lời, nhận diện giới tính và giọng vùng miền là những phần khó của một trợ lý ngân hàng chạy thật.
- **DevOps Agent / AIOps (Cloud Kinetic).** Một agent tự động điều tra sự cố: học ra topology của hệ thống, rồi phân loại, điều tra root cause, và đề xuất (không tự thực thi) phương án sửa. Demo trực tiếp mô phỏng tấn công DDoS đẩy latency từ 6s lên 12s; agent truy ra nguyên nhân là 1000 request/giây dồn vào load balancer và gợi ý các bước khắc phục. Kết quả ghi nhận qua các case: giảm 75-77% thời gian xử lý sự cố.
- **Amazon Quick cho HR (Noventiq).** Trợ lý agentic sàng lọc CV theo mô tả công việc, dựng một skill đánh giá dùng lại được, soạn JD, và xuất báo cáo chấm điểm ứng viên. Điều rút ra cho ứng viên: CV ngày càng được sàng lọc bằng AI, nên viết CV bám sát mô tả công việc.
- **Amazon Quick với MCP private (Renova Cloud).** Cách kết nối Quick tới một MCP server bên thứ ba mà không đi qua internet công khai: đặt MCP server trong private subnet, truy cập qua VPC connection với DNS resolver nội bộ và một ALB xử lý TLS, xác thực bằng Cognito. Các đầu mục chi phí (Route 53 resolver, ALB, data transfer) được nói thẳng.

### Điều em học được
Điểm lặp lại ở cả năm phần là AI hỗ trợ kỹ sư chứ không thay thế: DevOps agent chỉ đề xuất, trợ lý voice chuyển cuộc gọi khó cho người thật, công cụ HR vẫn để con người ra quyết định. Điều này khớp với hướng đồ án của em, nơi lớp AI phân tích và trả lời còn người dùng vẫn giữ quyền quyết định. Phần MCP private là phần hữu ích nhất về mặt kỹ thuật. Nó cho thấy "nối AI vào một dịch vụ bên thứ ba" chưa phải toàn bộ công việc trong môi trường doanh nghiệp: giữ kết nối đó nằm ngoài internet công khai, có DNS, TLS và xác thực riêng, mới là thứ khiến một ngân hàng chấp nhận. Nó cũng cho em một hình dung rõ hơn về hướng nghề ngoài lập trình, khi hai diễn giả làm ở vai trò solution và sale dựng trên cùng nền kiến thức cloud.

### Hình ảnh
[Chèn 1-2 ảnh hoặc liên kết video làm bằng chứng.]
