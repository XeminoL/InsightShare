---
title: "Clean up"
date: 2026-07-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Delete what was created

These resources keep incurring charges, so delete them once the project has been graded. `cleanup-aws.ps1 -Force` runs everything below except the CloudFront distribution, which has to be disabled first.

```bash
aws apigatewayv2 delete-api --api-id <api-id>

aws lambda delete-function --function-name insightshare-api

aws dynamodb delete-table --table-name insightshare-files

aws s3 rm s3://insightshare-files-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-files-khang-2352464
aws s3 rm s3://insightshare-web-khang-2352464 --recursive
aws s3api delete-bucket --bucket insightshare-web-khang-2352464

aws cognito-idp delete-user-pool-client --user-pool-id <user-pool-id> --client-id <app-client-id>
aws cognito-idp delete-user-pool --user-pool-id <user-pool-id>

aws cloudwatch delete-alarms --alarm-names insightshare-lambda-errors insightshare-lambda-throttles
aws cloudwatch delete-dashboards --dashboard-names insightshare-monitoring
aws logs delete-log-group --log-group-name /aws/lambda/insightshare-api

aws iam delete-role-policy --role-name insightshare-lambda-role --policy-name insightshare-lambda-permissions
aws iam delete-role --role-name insightshare-lambda-role
```

The Cognito user pool holds the registered accounts, so deleting it removes that data along with the rest of the stack.

Versioning is on for the file bucket, so `aws s3 rm --recursive` leaves the older object versions and the delete markers behind and `delete-bucket` then fails with `BucketNotEmpty`. `cleanup-aws.ps1` deletes every version and delete marker first, which is why the script is the easier path here.

{{% notice warning %}}
**CloudFront takes two steps.** A distribution cannot be deleted while it is enabled: set `Enabled` to `false`, wait for the status to return to `Deployed`, then delete it with the current ETag. This is the one resource the script reports instead of removing.
{{% /notice %}}

```bash
aws cloudfront get-distribution-config --id <dist-id> > dist.json
# keep only the DistributionConfig object from dist.json, set "Enabled": false,
# save it as config.json, then:
aws cloudfront update-distribution --id <dist-id> \
  --distribution-config file://config.json --if-match <etag>
# wait until Deployed, then:
aws cloudfront delete-distribution --id <dist-id> --if-match <new-etag>
```