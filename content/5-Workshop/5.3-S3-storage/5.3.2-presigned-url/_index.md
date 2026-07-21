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

Lambda creates the S3 client and generates a PUT URL for uploads and a GET URL for downloads, expiring after 15 minutes (`PRESIGN_EXPIRY = 900`).

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
**Technical note.** The default S3 client uses the global endpoint (`s3.amazonaws.com`), which returns **HTTP 307** when uploading to a bucket in Singapore. Pinning the regional endpoint plus Signature V4 (as above) makes the upload return HTTP 200.
{{% /notice %}}

#### Step 2: Test upload through the presigned URL

Request an upload URL, then PUT a file to it:

```bash
curl -X POST "$API/files" -H "Content-Type: application/json" \
  -d '{"filename":"test.txt","content_type":"text/plain"}'

curl -X PUT "<upload_url>" -H "Content-Type: text/plain" \
  --data-binary @test.txt -w "HTTP %{http_code}\n"
```

#### Step 3: Check the object in S3

Confirm the file landed in the bucket:

```bash
aws s3 ls s3://insightshare-files-khang-2352464/ --recursive
```

![Console: uploaded object in the S3 bucket](/images/5-Workshop/5.3-S3-storage/s3-object-uploaded.png)

_Screenshot: your AWS Console showing the uploaded object under its `{file_id}/` prefix in the bucket (screenshot to add)._
