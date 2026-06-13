---
id: apigw-002
title: API Gateway Deployments Don't Auto-Update
services: [API Gateway]
summary: AWS::ApiGateway::Deployment is immutable; new methods 403/404 until you force a redeploy.
---

# API Gateway Deployments Don't Auto-Update

**Problem:** Added new methods to API Gateway via CloudFormation, stack updated successfully, but new endpoints returned 403/404.

**Root Cause:** `AWS::ApiGateway::Deployment` is immutable. CloudFormation only creates a new deployment when the deployment resource itself changes. Adding methods via DependsOn doesn't trigger redeployment.

**Solution:** After updating Lambda/API Gateway stacks, force a new deployment:
```bash
aws apigateway create-deployment \
  --rest-api-id YOUR_API_ID \
  --stage-name dev \
  --description "Deploy new methods"
```

Or add a timestamp/hash to force CloudFormation to recreate the deployment:
```yaml
ApiDeployment:
  Type: AWS::ApiGateway::Deployment
  Properties:
    Description: !Sub 'Deployment ${AWS::StackName}-${Timestamp}'
```
