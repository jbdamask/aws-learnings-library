---
id: apigw-004
title: Lambda::Permission Required Per-Function for API Gateway
services: [API Gateway, Lambda]
summary: Each Lambda invoked by API Gateway needs its own AWS::Lambda::Permission, or you get 500 with zero Lambda logs.
---

# Lambda::Permission Required Per-Function for API Gateway

**Problem:** New API Gateway endpoint returned 500 Internal Server Error. Lambda CloudWatch logs showed zero invocations.

**Root Cause:** Each Lambda function invoked by API Gateway needs its own `AWS::Lambda::Permission` resource. Without it, API Gateway gets "Access Denied" when trying to invoke the Lambda. The error isn't visible in the Lambda's logs because the invocation is blocked at the API Gateway → Lambda boundary.

**Solution:** Add a Lambda permission for every Lambda function that API Gateway calls:
```yaml
MyFunctionPermission:
  Type: AWS::Lambda::Permission
  Properties:
    FunctionName: !Ref MyFunction
    Action: lambda:InvokeFunction
    Principal: apigateway.amazonaws.com
    SourceArn: !Sub 'arn:aws:execute-api:${AWS::Region}:${AWS::AccountId}:${ApiId}/*/*/*'
```

**Key Insight:** When an API Gateway endpoint returns 500 and the Lambda has no logs at all, the first thing to check is whether the `Lambda::Permission` exists.
