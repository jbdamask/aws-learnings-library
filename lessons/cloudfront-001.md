---
id: cloudfront-001
title: CloudFront Compression Requires Caching
services: [CloudFront]
summary: EnableAcceptEncodingGzip is invalid on a cache policy with caching disabled (all TTLs 0); set it false for no-cache policies.
---

# CloudFront Compression Requires Caching

**Problem:** CloudFront stack failed with "EnableAcceptEncodingGzip is not valid for CachePolicyConfig with caching disabled".

**Root Cause:** Can't enable compression on cache policies that have `MinTTL: 0, MaxTTL: 0, DefaultTTL: 0`.

**Solution:** Disable compression for no-cache policies:
```yaml
IndexCachePolicy:
  Type: AWS::CloudFront::CachePolicy
  Properties:
    CachePolicyConfig:
      MinTTL: 0
      MaxTTL: 0
      DefaultTTL: 0
      ParametersInCacheKeyAndForwardedToOrigin:
        EnableAcceptEncodingGzip: false  # Must be false when caching disabled
```
