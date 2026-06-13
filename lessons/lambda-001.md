---
id: lambda-001
title: Lambda Handler Paths Must Match Build Output Structure
services: [Lambda]
summary: esbuild outputs to subdirectories; the CloudFormation Handler must include the directory prefix or you get "Handler not found".
---

# Lambda Handler Paths Must Match Build Output Structure

**Problem:** Lambda returned "Handler not found" errors.

**Root Cause:** esbuild outputs files to subdirectories (e.g., `auth/initiateOAuth.mjs`), but CloudFormation handler was set to just `initiateOAuth.handler`.

**Solution:** Include the directory prefix in the handler path:
```yaml
Handler: auth/initiateOAuth.handler  # NOT just initiateOAuth.handler
```
