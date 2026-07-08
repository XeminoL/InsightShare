---
title: "Worklog Tuần 3"
date: 2026-06-15
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3

* Dựng ứng dụng web InsightShare chạy ở local.
* Tách lớp lưu trữ và lớp AI để sau dễ thay thế.
* Kiểm thử luồng cơ bản trước khi đưa lên cloud.

### Các công việc trong tuần (15/06 - 19/06/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Khởi tạo dự án FastAPI với virtualenv và `uvicorn`; thiết kế endpoint upload và liệt kê file (POST /files, GET /files) cùng schema request/response. | 15/06/2026 | 15/06/2026 | [FastAPI](https://fastapi.tiangolo.com/) |
| 3 | Viết API upload và liệt kê file, tạm lưu vào thư mục local và trả về id, tên, dung lượng file. | 16/06/2026 | 16/06/2026 |  |
| 4 | Dựng frontend tĩnh (HTML/JS, dùng fetch): form chọn file, danh sách file và thanh tiến trình upload. | 17/06/2026 | 17/06/2026 |  |
| 5 | Tách lớp lưu trữ và lớp AI thành interface trong Python (abstract base class) để sau lắp bản cloud vào; tạm mock kết quả AI. | 18/06/2026 | 18/06/2026 |  |
| 6 | Refactor theo lớp (API / service / storage) và viết unit test bằng pytest cho phần xử lý file. | 19/06/2026 | 19/06/2026 |  |

### Kết quả đạt được

1. Có ứng dụng chạy được ở local: upload và liệt kê file qua giao diện.
2. Lớp lưu trữ và lớp AI đã tách rõ, sẵn sàng thay bản thật ở các tuần sau.
3. Code tổ chức theo lớp, có unit test cho phần lõi.
