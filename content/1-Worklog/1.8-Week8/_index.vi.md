---
title: "Worklog Tuần 8"
date: 2026-07-20
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8

* Thêm đăng nhập người dùng bằng Amazon Cognito.
* Tách dữ liệu file theo user để mỗi người chỉ thấy file của mình.

### Các công việc trong tuần (20/07 - 24/07/2026)

| Thứ | Công việc | Bắt đầu | Hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Tạo Cognito user pool và app client với USER_PASSWORD_AUTH; ghi lại pool id, client id và token endpoint. | 20/07/2026 | 20/07/2026 | [Cognito lab](https://000081.awsstudygroup.com) |
| 3 | Thêm JWT authorizer vào route `$default` của API Gateway; thêm route OPTIONS không xác thực để CORS preflight không bị chặn bằng 401. | 21/07/2026 | 21/07/2026 |  |
| 4 | Đọc claim `sub` từ JWT trong Lambda (hàm `current_user`) để biết user gọi và lọc file theo owner. | 22/07/2026 | 22/07/2026 |  |
| 5 | Nối frontend gọi thẳng cognito-idp cho đăng ký, xác nhận và đăng nhập, không dùng thư viện ngoài. | 23/07/2026 | 23/07/2026 |  |
| 6 | Kiểm thử luồng: đăng ký → xác nhận → đăng nhập → gọi API kèm token (200), không token (401), và OPTIONS (200). | 24/07/2026 | 24/07/2026 | Postman |

### Kết quả đạt được

1. Người dùng đăng ký, xác nhận và đăng nhập qua Cognito; API chỉ nhận request có JWT hợp lệ.
2. Lambda đọc claim `sub` để biết user gọi và trả về file đã lọc theo owner, nên mỗi người chỉ thấy file của mình.
3. Route OPTIONS giữ không xác thực nên CORS preflight qua được, còn các route khác đều cần token.
