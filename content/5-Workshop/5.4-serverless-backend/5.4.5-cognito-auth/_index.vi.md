---
title: "Đăng nhập theo người dùng với Amazon Cognito"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Mục tiêu

Thêm đăng nhập để mỗi người chỉ thấy file của chính mình. **Amazon Cognito** giữ danh bạ người dùng, **API Gateway** kiểm tra token bằng JWT authorizer, và **Lambda** đọc claim `sub` để gán mỗi file cho đúng chủ sở hữu.

#### Bước 1: Tạo user pool và app client

Tạo một Cognito user pool và một app client. User pool là danh bạ tài khoản; app client là cách web app này tự khai báo với pool. App client tạo ra không có client secret và bật luồng `USER_PASSWORD_AUTH`, nên một trang tĩnh gọi thẳng API của Cognito từ trình duyệt được: không SDK, không server, không có secret phải giữ. `--auto-verified-attributes email` để Cognito gửi mã xác nhận qua email khi đăng ký.

```bash
aws cognito-idp create-user-pool \
  --pool-name insightshare-users \
  --auto-verified-attributes email

aws cognito-idp create-user-pool-client \
  --user-pool-id <user-pool-id> \
  --client-name insightshare-web \
  --no-generate-secret \
  --explicit-auth-flows ALLOW_USER_PASSWORD_AUTH ALLOW_REFRESH_TOKEN_AUTH
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

Vì token đã được kiểm tra ở phía trên, Lambda chỉ cần đọc người gọi là ai. Nó lấy claim `sub`, là id duy nhất và ổn định của tài khoản Cognito, và trả về chuỗi `public` nếu không có claim nào:

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

Mỗi lần upload lưu `user_id = current_user(event)`. Liệt kê, tìm kiếm và hỏi đáp thư viện chỉ trả về file của chính người gọi; một file chỉ chủ của nó mới đọc, phân tích, hỏi hay xóa được.

Đường lùi `public` chỉ có ý nghĩa khi thiếu authorizer. Trên API đã deploy, authorizer nằm ở route `$default`, nên request không có token bị chặn ngay tại gateway: gọi `curl` vào `/files` mà không có header `Authorization` trả về `401 Unauthorized`, hàm không hề chạy.

#### Bước 4: Form đăng nhập ở frontend

Frontend vẫn là một file HTML tự chứa, không script bên ngoài, không SDK. Hai hằng số giữ `COGNITO_REGION` và `COGNITO_CLIENT_ID`; form đăng ký, xác nhận mã và đăng nhập nằm ngay trong trang, gửi thẳng tới endpoint API của Cognito theo region, mỗi thao tác một `X-Amz-Target`:

```javascript
const COGNITO_REGION = "ap-southeast-1";
const COGNITO_CLIENT_ID = "<app-client-id>";

async function cognito(action, payload){
  const r = await fetch("https://cognito-idp." + COGNITO_REGION + ".amazonaws.com/", {
    method: "POST",
    headers: {
      "Content-Type": "application/x-amz-json-1.1",
      "X-Amz-Target": "AWSCognitoIdentityProviderService." + action,
    },
    body: JSON.stringify(payload),
  });
  const d = await r.json();
  if (!r.ok) throw new Error(d.message || action + " failed");
  return d;
}
```

Đăng ký gọi `SignUp`, mã gửi qua email đưa vào `ConfirmSignUp`, đăng nhập gọi `InitiateAuth` với `AuthFlow: "USER_PASSWORD_AUTH"`. `IdToken` trong response được giữ trong `localStorage` và gửi kèm `Authorization: Bearer <id_token>` ở mọi lời gọi API. Khi API Gateway trả 401, trang hiện lại form đăng nhập.

```javascript
const d = await cognito("InitiateAuth", {
    ClientId: COGNITO_CLIENT_ID,
    AuthFlow: "USER_PASSWORD_AUTH",
    AuthParameters: { USERNAME: email, PASSWORD: pass },
});
localStorage.setItem("insightshare_id_token", d.AuthenticationResult.IdToken);
```

{{% notice note %}}
**Vì sao dùng form thay vì Hosted UI của Cognito.** Hosted UI là lựa chọn phổ biến hơn và giữ mật khẩu nằm ngoài trang ứng dụng, nhưng nó cần một Cognito domain kèm callback URL khai báo cho từng môi trường, và nó chuyển hướng ra khỏi app rồi quay lại. Gọi API trực tiếp giữ toàn bộ luồng trong một file HTML tĩnh, đúng như frontend này. Đánh đổi là `USER_PASSWORD_AUTH` gửi mật khẩu từ trình duyệt tới Cognito, nên phải dựa vào app client không có secret và vào HTTPS.
{{% /notice %}}
