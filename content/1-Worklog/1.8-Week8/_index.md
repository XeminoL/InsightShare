---
title: "Week 8 Worklog"
date: 2026-07-20
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 objectives

* Add user sign-in with Amazon Cognito.
* Scope file data per user so each person only sees their own files.

### Tasks during the week (20/07 - 24/07/2026)

| Day | Task | Start | End | Reference |
| --- | --- | --- | --- | --- |
| Mon | Create a Cognito user pool and an app client with USER_PASSWORD_AUTH; note the pool id, client id and token endpoint. | 20/07/2026 | 20/07/2026 | [Cognito lab](https://000081.awsstudygroup.com) |
| Tue | Add a JWT authorizer to the API Gateway `$default` route; add an unauthenticated OPTIONS route so the CORS preflight is not blocked with 401. | 21/07/2026 | 21/07/2026 |  |
| Wed | Read the `sub` claim from the JWT in Lambda (a `current_user` helper) to identify the caller and filter files by owner. | 22/07/2026 | 22/07/2026 |  |
| Thu | Wire the frontend to call cognito-idp directly for sign-up, confirm and sign-in, without an external library. | 23/07/2026 | 23/07/2026 |  |
| Fri | Test the flow: sign-up → confirm → sign-in → call the API with the token (200), without a token (401), and OPTIONS (200). | 24/07/2026 | 24/07/2026 | Postman |

### Results achieved

1. Users can sign up, confirm and sign in through Cognito; the API accepts requests only with a valid JWT.
2. Lambda reads the `sub` claim to know the caller and returns files filtered by owner, so each user sees only their own.
3. The OPTIONS route stays unauthenticated, so the CORS preflight passes while other routes require a token.
