---
title: "Per-user sign-in with Amazon Cognito"
date: 2026-07-29
weight: 5
chapter: false
pre: " <b> 5.4.5 </b> "
---

#### Goal

Add sign-in so each user sees only their own files. **Amazon Cognito** provides the user directory and a Hosted UI login page, **API Gateway** validates the token with a JWT authorizer, and the **Lambda** reads the `sub` claim to scope every file to its owner. The design is fail-open: when Cognito is not configured the application keeps working as a single shared library, so it can be rolled out incrementally.

#### Step 1: Create a user pool and app client

Create a Cognito user pool, an app client, and a Hosted UI domain. The app client uses the OAuth2 implicit flow so the static frontend needs no SDK and no backend secret.

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

_Screenshot: your AWS Console showing the `insightshare-users` user pool with the app client and Hosted UI domain (screenshot to add)._

#### Step 2: Add a JWT authorizer on API Gateway

Attach a JWT authorizer to the HTTP API. Its issuer is the user pool and its audience is the app client, so API Gateway verifies the token signature and expiry before the request reaches Lambda.

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

_Screenshot: your AWS Console showing the `cognito-jwt` authorizer attached to the API routes (screenshot to add)._

#### Step 3: Lambda scopes data per user

The Lambda derives the current user from the `sub` claim and falls back to `public` when no authorizer is present:

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

Each upload stores `user_id = current_user(event)`. List, search and library Q&A return only the caller's items, and a single file is readable, analyzable, askable or deletable only by its owner. Items saved before this change carry no `user_id` and stay visible to everyone as legacy public data, so existing files do not disappear.

#### Step 4: Frontend sign-in through the Hosted UI

The frontend stays a single self-contained HTML file with no external script. Three constants near the API constant hold `COGNITO_DOMAIN`, `COGNITO_CLIENT_ID` and `COGNITO_REDIRECT`. When they are empty the app runs anonymously as before. When they are set, a **Sign in** button redirects to the Hosted UI; on return the page reads the `id_token` from the URL fragment, stores it, shows the signed-in email with a **Sign out** button, and sends `Authorization: Bearer <id_token>` on every API call.

{{% notice info %}}
**Fail-open by design.** With `USER_POOL_ID` unset on the Lambda and `COGNITO_DOMAIN` empty on the frontend, no token is sent, `current_user` returns `public`, and the application behaves exactly as it did before authentication was added. This lets sign-in be enabled in one step without a code change.
{{% /notice %}}
