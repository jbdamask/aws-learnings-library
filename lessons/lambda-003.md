---
id: lambda-003
title: S3 Lambda Zips Can Go Stale — Deploy Script Must Update Function Code
services: [Lambda, S3]
summary: Uploading a new zip to S3 doesn't update running Lambdas; you must explicitly update each function's code.
---

# S3 Lambda Zips Can Go Stale — Deploy Script Must Update Function Code

**Problem:** Logout Lambda returned 502 with `Cannot find module 'logout'`, even though the handler path was correct and the code was committed to git.

**Root Cause:** The `auth.zip` on S3 was uploaded before `logout.ts` existed. CloudFormation created the Lambda pointing to `auth.zip`, but the zip only had 3 of 4 auth handlers. Uploading a new zip to S3 doesn't automatically update running Lambdas — they cache the code from their last deployment.

**Solution:** After uploading new code to S3, explicitly update each Lambda's function code:
```bash
aws lambda update-function-code \
  --function-name myapp-logout-dev \
  --s3-bucket myapp-deployments-us-east-1 \
  --s3-key lambdas/dev/auth.zip
```

**Better Solution:** The deploy script should do this automatically after uploading zips. Upload to S3 is not deployment — updating the Lambda function code is.

**Key Insight:** "Code in git" ≠ "code on S3" ≠ "code running in Lambda." All three must be in sync. Verify with behavioral tests against live endpoints, not just `git status` or S3 listings.
