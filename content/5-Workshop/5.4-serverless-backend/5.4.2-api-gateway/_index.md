---
title: "Create the API Gateway"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

#### Goal

Create an **API Gateway (HTTP API)** as the public entry point through which the frontend calls Lambda. HTTP API is cheaper and simpler than REST API and is enough here.

#### Create the API

Because the Lambda already dispatches by method and path internally, a single **`$default`** route forwarding everything to Lambda is all that is needed:

```bash
aws apigatewayv2 create-api \
  --name insightshare-api \
  --protocol-type HTTP \
  --target arn:aws:lambda:ap-southeast-1:<account-id>:function:insightshare-api \
  --cors-configuration "AllowOrigins=*,AllowMethods=GET,POST,DELETE,OPTIONS,AllowHeaders=*"
# -> returns ApiId and ApiEndpoint (the invoke URL)
```

Using `--target` auto-creates the Lambda integration, the `$default` route and the `$default` stage (auto-deploy). Then grant API Gateway permission to invoke the Lambda:

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

#### Routes handled by the Lambda

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

#### Test the API

```bash
API="https://<api-id>.execute-api.ap-southeast-1.amazonaws.com"

# POST /files -> presigned URL
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"hello.txt","content_type":"text/plain"}'

# GET /files -> list
curl "$API/files"
```

{{% notice info %}}
**Technical note.** DynamoDB returns numbers (such as `uploaded_at`) as Python `Decimal`, which `json.dumps` cannot serialize, producing `Object of type Decimal is not JSON serializable`. A custom JSON encoder converts `Decimal` to `int`/`float` and is used in every response.

```python
class _DecimalEncoder(json.JSONEncoder):
    def default(self, o):
        if isinstance(o, decimal.Decimal):
            return int(o) if o % 1 == 0 else float(o)
        return super().default(o)
# ... json.dumps(body, cls=_DecimalEncoder)
```
{{% /notice %}}

After the fix, `GET /files` and `GET /files/search?q=` both return clean JSON.
