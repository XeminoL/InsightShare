---
title: "Create the S3 bucket & configure it"
date: 2026-07-29
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

#### Step 1: Create the S3 bucket

This bucket is the single place every uploaded file is stored; the Lambda later reads and writes objects here and issues presigned URLs against it. Open the **S3 console** (region `ap-southeast-1`) and choose **Create bucket**:

- **Bucket name**: `insightshare-files-khang-2352464` (S3 names are globally unique, so a personal suffix avoids collisions).
- **Region**: Asia Pacific (Singapore) `ap-southeast-1`, the same region as the Lambda so requests stay in-region and presigning uses the regional endpoint.

After creating it, the bucket opens empty:

![S3 bucket created](/images/5-Workshop/5.3-S3-storage/s3-bucket-created.png)

Verify the region from the CLI:

```bash
aws s3api get-bucket-location --bucket insightshare-files-khang-2352464
```

#### Step 2: Block public access

The bucket holds user files that must never be world-readable; access is granted per object and per request through presigned URLs, so the bucket itself needs no public access. Keep **all four Block Public Access boxes ticked**: the four settings block public ACLs, ignore existing public ACLs, block public bucket policies, and restrict cross-account public policies, so no combination of ACL or policy can accidentally expose an object.

![Console: S3 Block Public Access on](/images/5-Workshop/5.3-S3-storage/s3-block-public-access.png)

Confirm the same state from the CLI:

```bash
aws s3api get-public-access-block --bucket insightshare-files-khang-2352464
```

#### Step 3: Enable versioning

Versioning keeps a prior copy of an object when it is overwritten or deleted, so an accidental re-upload to the same key or a wrong delete can be recovered instead of losing the file. Enable **Bucket Versioning**:

![Console: S3 versioning enabled](/images/5-Workshop/5.3-S3-storage/s3-versioning.png)

Confirm it from the CLI:

```bash
aws s3api get-bucket-versioning --bucket insightshare-files-khang-2352464
```

#### Step 4: Configure CORS

The browser uploads and downloads files directly to S3 through presigned URLs, and those requests come from the web page origin, not from S3's own domain, so S3 rejects them unless CORS explicitly allows that origin. `AllowedMethods` lists `PUT` for uploads and `GET` for downloads, the only two verbs presigned URLs use here; `ExposeHeaders` returns `ETag` so the browser can read the upload's object hash; `MaxAgeSeconds` caches the CORS preflight for 3000 seconds to avoid an extra round trip per request. Apply this CORS configuration (demo scale allows any origin; in production restrict `AllowedOrigins` to your web domain):

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

- Objects are stored under the prefix `{file_id}/{filename}`, so each file lives in its own folder keyed by a unique id, which is also the DynamoDB item key that ties the object to its metadata.
- The Lambda function reads the bucket name `insightshare-files-khang-2352464` from its `BUCKET` environment variable, so the code carries no hard-coded bucket name.
