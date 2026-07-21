---
title: "Generate & test the presigned URL"
date: 2026-07-29
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### What is a presigned URL

A **presigned URL** is a signed link to one S3 object, valid for a limited time, so the browser uploads/downloads directly while the bucket stays private.

#### Step 1: Generate a presigned URL with boto3

The presigned URL carries the Lambda role's signature in its query string, so the browser can call S3 directly for one specific object and verb without any AWS credentials of its own. A short 15-minute expiry (`PRESIGN_EXPIRY = 900`) limits how long a leaked link stays usable. The client pins the regional endpoint and Signature V4 for the reason in the technical note below.

```python
from botocore.config import Config

s3 = boto3.client(
    "s3",
    region_name="ap-southeast-1",
    endpoint_url="https://s3.ap-southeast-1.amazonaws.com",
    config=Config(signature_version="s3v4", s3={"addressing_style": "virtual"}),
)

put_url = s3.generate_presigned_url(
    "put_object",
    Params={"Bucket": BUCKET, "Key": key, "ContentType": content_type},
    ExpiresIn=900,
)

get_url = s3.generate_presigned_url(
    "get_object",
    Params={"Bucket": BUCKET, "Key": key},
    ExpiresIn=900,
)
```

{{% notice info %}}
**Technical note.** The default S3 client signs against the global endpoint (`s3.amazonaws.com`). A bucket in `ap-southeast-1` answers that endpoint with an **HTTP 307 Temporary Redirect** to its regional host, and the browser drops the signed headers on the redirect, so the `PUT` fails. Setting `endpoint_url` to `https://s3.ap-southeast-1.amazonaws.com` makes S3 answer directly with no redirect, and `signature_version="s3v4"` signs with SigV4, which the regional endpoint requires. Together they make the upload return HTTP 200 on the first request.
{{% /notice %}}

#### Step 2: Test upload through the presigned URL

This is the two-call upload the browser performs: `POST /files` asks the Lambda for a signed URL, then a single `PUT` sends the bytes straight to S3. The `-w "HTTP %{http_code}"` prints the status so the 200 is visible.

```bash
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"test.txt","content_type":"text/plain"}'

curl -X PUT "<upload_url>" -H "Content-Type: text/plain" \
  --data-binary @test.txt -w "HTTP %{http_code}\n"
```

#### Step 3: Check the object in S3

The object appears under its `{file_id}/{filename}` prefix, which proves the presigned PUT reached the private bucket without the bucket being public. This uploaded object is what the AI layer later reads to produce labels and text.

```bash
aws s3 ls s3://insightshare-files-khang-2352464/ --recursive
```

![Console: uploaded object in the S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-object-uploaded.png)

