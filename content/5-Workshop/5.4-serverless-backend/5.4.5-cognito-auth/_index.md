---
title: "Per-user sign-in with Amazon Cognito"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Goal

Add sign-in so each user sees only their own files. **Amazon Cognito** holds the user directory, **API Gateway** validates the token with a JWT authorizer, and the **Lambda** reads the `sub` claim to scope every file to its owner.

#### Step 1: Create a user pool and app client

Create a Cognito user pool and an app client. The user pool is the directory of accounts; the app client represents this web app to the pool. The client is created with no client secret and with the `USER_PASSWORD_AUTH` flow enabled, so a static page can call the Cognito API directly from the browser: no SDK, no server, and no secret to keep. `--auto-verified-attributes email` makes Cognito email a confirmation code on sign-up.

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

![Console: the Cognito user pool](/images/5-Workshop/5.4-serverless-backend/cognito-user-pool.png)

The pool in the screenshot was created through the Console first, so it kept the name the Console generated. The pool id is what the rest of the setup refers to, not the display name.

#### Step 2: Add a JWT authorizer on API Gateway

The authorizer moves token checking out of the Lambda and into the gateway, so an unauthenticated or expired token is rejected with 401 before any code runs. Attach a JWT authorizer to the HTTP API: its `Issuer` is the user pool and its `Audience` is the app client, and API Gateway verifies the token signature and expiry on every request.

```bash
aws apigatewayv2 create-authorizer \
  --api-id <api-id> \
  --name cognito-jwt \
  --authorizer-type JWT \
  --identity-source '$request.header.Authorization' \
  --jwt-configuration Issuer=https://cognito-idp.ap-southeast-1.amazonaws.com/<user-pool-id>,Audience=<app-client-id>
```

The verified claims are placed at `event["requestContext"]["authorizer"]["jwt"]["claims"]`, which the Lambda reads. Because API Gateway already verifies the signature, the Lambda does not re-verify it.

![Console: JWT authorizer on the HTTP API](/images/5-Workshop/5.4-serverless-backend/cognito-jwt-authorizer.png)

#### Step 3: Lambda scopes data per user

With the token verified upstream, the Lambda only needs to read who the caller is. It takes the `sub` claim, the stable unique id of the Cognito account, and falls back to the string `public` if no claims are present:

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

Each upload stores `user_id = current_user(event)`. List, search and library Q&A return only the caller's items, and a file can be read, analyzed, asked about or deleted only by its owner.

The `public` fallback only matters if the authorizer is missing. On the deployed API the authorizer sits on the `$default` route, so a request without a token is rejected at the gateway: `curl` against `/files` with no `Authorization` header returns `401 Unauthorized` and the function is never invoked.

#### Step 4: Frontend sign-in form

The frontend stays a single self-contained HTML file with no external script and no SDK. Two constants hold `COGNITO_REGION` and `COGNITO_CLIENT_ID`; the sign-up, confirm and sign-in form lives in the page itself and posts straight to the Cognito API endpoint for the region, one `X-Amz-Target` per operation:

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

Sign-up calls `SignUp`, the emailed code goes to `ConfirmSignUp`, and sign-in calls `InitiateAuth` with `AuthFlow: "USER_PASSWORD_AUTH"`. The `IdToken` from the response is kept in `localStorage` and sent as `Authorization: Bearer <id_token>` on every API call. A 401 from API Gateway is what tells the page to show the form again.

```javascript
const d = await cognito("InitiateAuth", {
    ClientId: COGNITO_CLIENT_ID,
    AuthFlow: "USER_PASSWORD_AUTH",
    AuthParameters: { USERNAME: email, PASSWORD: pass },
});
localStorage.setItem("insightshare_id_token", d.AuthenticationResult.IdToken);
```

{{% notice note %}}
**Why a form instead of the Cognito Hosted UI.** The Hosted UI is the more common choice and it keeps the password off the application page, but it needs a Cognito domain plus callback URLs registered per environment, and it redirects away from the app and back. Calling the API directly keeps the whole flow inside one static HTML file, which is what this frontend is. The trade-off is that `USER_PASSWORD_AUTH` sends the password from the browser to Cognito, so it relies on the app client having no secret and on HTTPS.
{{% /notice %}}