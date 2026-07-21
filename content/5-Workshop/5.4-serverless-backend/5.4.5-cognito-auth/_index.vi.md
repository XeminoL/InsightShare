---
title: "Đăng nhập theo người dùng với Amazon Cognito"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Mục tiêu

Thêm đăng nhập để mỗi người chỉ thấy file của chính mình. **Amazon Cognito** cung cấp user directory và trang đăng nhập Hosted UI, **API Gateway** kiểm tra token bằng JWT authorizer, và **Lambda** đọc claim `sub` để gán mỗi file cho đúng chủ sở hữu. Thiết kế theo hướng fail-open: khi chưa cấu hình Cognito, ứng dụng vẫn chạy như một thư viện dùng chung, nên có thể bật dần từng bước.

#### Bước 1: Tạo user pool và app client

Tạo một Cognito user pool, một app client và một Hosted UI domain. User pool là danh bạ tài khoản; app client đại diện cho web app này với pool; Hosted UI domain là trang đăng nhập do AWS phục vụ. App client dùng OAuth2 implicit flow với scope `openid email profile`, nên token được trả thẳng về trình duyệt trong URL fragment và frontend tĩnh không cần SDK, không cần server, không cần client secret để hoàn tất đăng nhập.

```bash
aws cognito-idp create-user-pool \
  --pool-name insightshare-users \
  --auto-verified-attributes email

aws cognito-idp create-user-pool-client \
  --user-pool-id <user-pool-id> \
  --client-name insightshare-web \
  --allowed-o-auth-flows implicit \
  --allowed-o-auth-scopes openid email profile \
  --allowed-o-auth-flows-user-pool-client \
  --callback-urls "https://<cloudfront-domain>/" \
  --logout-urls "https://<cloudfront-domain>/" \
  --supported-identity-providers COGNITO

aws cognito-idp create-user-pool-domain \
  --domain insightshare-<suffix> \
  --user-pool-id <user-pool-id>
```

![Console: Cognito user pool](/images/5-Workshop/5.4-serverless-backend/cognito-user-pool.png)

#### Bước 2: Thêm JWT authorizer trên API Gateway

Authorizer đưa việc kiểm tra token ra khỏi Lambda và vào gateway, nên token chưa xác thực hay hết hạn bị từ chối 401 trước khi bất kỳ code nào chạy. Gắn một JWT authorizer vào HTTP API: `Issuer` là user pool (nên chỉ token do pool đó phát mới được nhận) và `Audience` là app client (nên chỉ token phát cho app này mới qua), và API Gateway kiểm tra chữ ký và hạn của token trong mọi request.

```bash
aws apigatewayv2 create-authorizer \
  --api-id <api-id> \
  --name cognito-jwt \
  --authorizer-type JWT \
  --identity-source '$request.header.Authorization' \
  --jwt-configuration Issuer=https://cognito-idp.ap-southeast-1.amazonaws.com/<user-pool-id>,Audience=<app-client-id>
```

Các claim đã kiểm tra được đặt tại `event["requestContext"]["authorizer"]["jwt"]["claims"]` để Lambda đọc. Vì API Gateway đã kiểm tra chữ ký, Lambda không kiểm tra lại.

![Console: JWT authorizer trên HTTP API](/images/5-Workshop/5.4-serverless-backend/cognito-jwt-authorizer.png)

#### Bước 3: Lambda gán dữ liệu theo người dùng

Vì token đã được kiểm tra ở phía trên, Lambda chỉ cần đọc người gọi là ai. Nó lấy người dùng hiện tại từ claim `sub` (id duy nhất ổn định của tài khoản Cognito), và trả về `public` khi không có authorizer, chính là thứ giữ hành vi fail-open:

```python
def current_user(event):
    try:
        claims = event["requestContext"]["authorizer"]["jwt"]["claims"]
        sub = claims.get("sub")
        if sub:
            return sub
    except (KeyError, TypeError):
        pass
    return "public"
```

Mỗi lần upload lưu `user_id = current_user(event)`. Liệt kê, tìm kiếm và hỏi đáp thư viện chỉ trả về file của chính người gọi, và một file chỉ được đọc, phân tích, hỏi hay xóa bởi đúng chủ của nó. File lưu trước khi có thay đổi này không có `user_id` nên được coi là dữ liệu public cũ và mọi người vẫn thấy, để file cũ không biến mất.

#### Bước 4: Đăng nhập ở frontend qua Hosted UI

Frontend vẫn là một file HTML tự chứa, không có script bên ngoài. Ba hằng số cạnh hằng số API giữ `COGNITO_DOMAIN`, `COGNITO_CLIENT_ID` và `COGNITO_REDIRECT`. Khi rỗng, ứng dụng chạy ẩn danh như trước. Khi được đặt, nút **Sign in** chuyển hướng tới Hosted UI; khi quay về, trang đọc `id_token` từ URL fragment, lưu lại, hiển thị email đã đăng nhập cùng nút **Sign out**, và gửi `Authorization: Bearer <id_token>` trong mọi lời gọi API.

{{% notice info %}}
**Fail-open theo thiết kế.** Khi `USER_POOL_ID` không đặt trên Lambda và `COGNITO_DOMAIN` để rỗng ở frontend, không có token nào được gửi, `current_user` trả về `public`, và ứng dụng chạy đúng như trước khi thêm xác thực. Nhờ vậy có thể bật đăng nhập trong một bước mà không phải sửa code.
{{% /notice %}}
