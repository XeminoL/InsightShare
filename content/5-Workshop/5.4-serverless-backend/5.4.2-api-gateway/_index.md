---
title: "Create the API Gateway"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### Goal

Create an **API Gateway (HTTP API)** as the public entry point through which the frontend calls Lambda. HTTP API is cheaper and simpler than REST API and is enough here.

#### Step 1: Create the API

API Gateway is the public HTTPS front door: it terminates TLS, applies CORS, and (once the authorizer is added in 5.4.5) checks the Cognito token before any request reaches Lambda. Because the Lambda already dispatches by method and path internally, a single **`$default`** route forwarding everything to Lambda is all that is needed, so routes do not have to be declared twice. The `--cors-configuration` here lets the browser call the API cross-origin, mirroring the S3 CORS in 5.3:

```bash
aws apigatewayv2 create-api \
  --name insightshare-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:ap-southeast-1:<account-id>:function:insightshare-api \
  --cors-configuration "AllowOrigins=*,AllowMethods=GET,POST,DELETE,OPTIONS,AllowHeaders=*"
```

Using `--target` auto-creates the Lambda integration, the `$default` route and the `$default` stage (auto-deploy). API Gateway still needs an explicit resource-based permission on the function before it may invoke it; the `--source-arn` scopes that permission to this one API so no other API can call the function:

```bash
aws lambda add-permission \
  --function-name insightshare-api \
  --statement-id apigw-invoke \
  --action lambda:InvokeFunction \
  --principal apigateway.amazonaws.com \
  --source-arn "arn:aws:execute-api:ap-southeast-1:<account-id>:<api-id>/*/*"
```

The Routes view shows the single `$default` route managed by API Gateway, integrated with the Lambda:

![API Gateway routes](/images/5-Workshop/5.4-serverless-backend/apigateway-routes.png)

The screenshot confirms the one `$default` route pointing at the Lambda integration.

#### Step 2: Routes handled by the Lambda

These are the logical endpoints the frontend uses; all arrive through the single `$default` route and are separated inside the Lambda. Together they form the pipeline: upload a file, analyze it, then list, search or ask over the results.

| Method | Path | Purpose |
|---|---|---|
| POST | `/files` | Request a presigned upload URL + create metadata |
| POST | `/files/{id}/analyze` | Run Rekognition/Textract on the uploaded object |
| POST | `/files/{id}/ask` | Ask a question about the document (Bedrock/Claude, Vietnamese) |
| POST | `/ask` | Ask across the whole library (Bedrock/Claude, with source files) |
| GET | `/files` | List all files |
| GET | `/files/search?q=` | Content search over labels + extracted text |
| GET | `/files/{id}` | One file's metadata + presigned download URL |
| DELETE | `/files/{id}` | Delete the object + metadata |

#### Step 3: Test the API

```bash
API="https://<api-id>.execute-api.ap-southeast-1.amazonaws.com"

curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"hello.txt","content_type":"text/plain"}'

curl "$API/files"
```

{{% notice info %}}
**Technical note.** The `boto3` DynamoDB resource returns every stored number (such as `uploaded_at` and `size`) as a Python `Decimal` to preserve precision. `json.dumps` has no rule for `Decimal`, so the first `GET /files` failed with `Object of type Decimal is not JSON serializable`. A custom JSON encoder converts `Decimal` to `int` (or `float` when it has a fractional part) and is passed to `json.dumps` in every response, so the API always returns valid JSON numbers.

```python
class _DecimalEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(o, decimal.Decimal):
            return int(o) if o % 1 == 0 else float(o)
        return super().default(o)
```
{{% /notice %}}

After the fix, `GET /files` and `GET /files/search?q=` both return clean JSON.
