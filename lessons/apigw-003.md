---
id: apigw-003
title: CORS Requires OPTIONS Methods
services: [API Gateway]
summary: Browsers send an OPTIONS preflight before cross-origin POST/PUT/DELETE; without an OPTIONS method API Gateway returns 403.
---

# CORS Requires OPTIONS Methods

**Problem:** Browser CORS preflight requests failed with 403.

**Root Cause:** Browsers send OPTIONS preflight requests before cross-origin POST/PUT/DELETE. Without an OPTIONS method configured, API Gateway rejects the preflight.

**Solution:** Add OPTIONS method with MOCK integration for every resource:
```yaml
ServersOptionsMethod:
  Type: AWS::ApiGateway::Method
  Properties:
    HttpMethod: OPTIONS
    AuthorizationType: NONE
    Integration:
      Type: MOCK
      RequestTemplates:
        application/json: '{"statusCode": 200}'
      IntegrationResponses:
        - StatusCode: '200'
          ResponseParameters:
            method.response.header.Access-Control-Allow-Headers: "'Content-Type,Authorization'"
            method.response.header.Access-Control-Allow-Methods: "'GET,POST,OPTIONS'"
            method.response.header.Access-Control-Allow-Origin: "'*'"
    MethodResponses:
      - StatusCode: '200'
        ResponseParameters:
          method.response.header.Access-Control-Allow-Headers: true
          method.response.header.Access-Control-Allow-Methods: true
          method.response.header.Access-Control-Allow-Origin: true
```
