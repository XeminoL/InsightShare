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

Tạo một Cognito user pool, một app client và một Hosted UI domain. App client dùng OAuth2 implicit flow nên frontend tĩnh không cần SDK và không cần secret ở backend.

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

_Ảnh chụp: AWS Console hiển thị user pool `insightshare-users` cùng app client và Hosted UI domain (ảnh sẽ bổ sung)._

#### Bước 2: Thêm JWT authorizer trên API Gateway

Gắn một JWT authorizer vào HTTP API. Issuer là user pool và audience là app client, nên API Gateway kiểm tra chữ ký và hạn của token trước khi request tới Lambda.

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

_Ảnh chụp: AWS Console hiển thị authorizer `cognito-jwt` gắn vào các route của API (ảnh sẽ bổ sung)._

#### Bước 3: Lambda gán dữ liệu theo người dùng

Lambda lấy người dùng hiện tại từ claim `sub`, và trả về `public` khi không có authorizer:

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
