---
id: apigw-001
title: API Gateway Methods vs Resources Are Different Things
services: [API Gateway]
summary: Resources define URL paths but you also need Methods with Lambda integrations, or you get "Missing Authentication Token".
---

# API Gateway Methods vs Resources Are Different Things

**Problem:** Created API Gateway resources (URL paths like `/auth/github`, `/servers`) but got "Missing Authentication Token" errors when calling them.

**Root Cause:** Resources define the URL structure, but you also need to create Methods (GET, POST, OPTIONS) with Lambda integrations attached. Without methods, the paths exist but nothing handles requests.

**Solution:** Always define both the resource AND the methods with their Lambda proxy integrations:
```yaml
# Resource defines the path
ServersResource:
  Type: AWS::ApiGateway::Resource
  Properties:
    PathPart: servers

# Method defines what happens when you call it
ServersGetMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    ResourceId: !Ref ServersResource
    HttpMethod: GET
    Integration:
      Type: AWS_PROXY
      Uri: !Sub 'arn:aws:apigateway:${AWS::Region}:lambda:path/2015-03-31/functions/${ListServersFunction.Arn}/invocations'
```
