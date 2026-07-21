---
title: "Create the S3 bucket & configure it"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Step 1: Create the S3 bucket

Open the **S3 console** (region `ap-southeast-1`) and choose **Create bucket**:

- **Bucket name**: `insightshare-files-khang-2352464` (S3 names are globally unique, so a personal suffix avoids collisions).
- **Region**: Asia Pacific (Singapore) `ap-southeast-1`.

After creating it, the bucket opens empty:

![S3 bucket created](/images/5-Workshop/5.3-S3-storage/s3-bucket-created.png)

Verify the region from the CLI:

```bash
aws s3api get-bucket-location --bucket insightshare-files-khang-2352464
```

#### Step 2: Block public access

Keep **all four Block Public Access boxes ticked** so the bucket stays private:

![Console: S3 Block Public Access on](/images/5-Workshop/5.3-S3-storage/s3-block-public-access.png)

```bash
aws s3api get-public-access-block --bucket insightshare-files-khang-2352464
```

#### Step 3: Enable versioning

Enable **Bucket Versioning** so previous versions of an object are kept:

![Console: S3 versioning enabled](/images/5-Workshop/5.3-S3-storage/s3-versioning.png)

```bash
aws s3api get-bucket-versioning --bucket insightshare-files-khang-2352464
```

#### Step 4: Configure CORS

The browser uploads and downloads files directly to S3 through presigned URLs. For that to work, the bucket must allow cross-origin requests. Apply this CORS configuration (demo scale allows any origin; in production restrict `AllowedOrigins` to your web domain):

```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "PUT"],
      "AllowedOrigins": ["*"],
      "ExposeHeaders": ["ETag"],
      "MaxAgeSeconds": 3000
    }
  ]
}
```

Apply and verify it from the CLI:

```bash
aws s3api put-bucket-cors --bucket insightshare-files-khang-2352464 \
  --cors-configuration file://cors.json

aws s3api get-bucket-cors --bucket insightshare-files-khang-2352464
```

#### Notes

- Objects are stored under the prefix `{file_id}/{filename}`, so each file lives in its own folder keyed by a unique id.
- The Lambda function reads the bucket name `insightshare-files-khang-2352464` from its `BUCKET` environment variable.
