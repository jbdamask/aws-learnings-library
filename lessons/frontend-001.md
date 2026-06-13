---
id: frontend-001
title: Trailing Slashes Cause Double-Slash URLs
services: [Frontend]
summary: A base URL ending in / plus a path starting with / yields // — strip the trailing slash from the base URL.
---

# Trailing Slashes Cause Double-Slash URLs

**Problem:** API calls went to `/dev//auth/github` (double slash).

**Root Cause:** `VITE_API_BASE_URL` ended with `/` and endpoint paths started with `/`.

**Solution:** Strip trailing slashes from base URLs:
```typescript
const API_BASE_URL = import.meta.env.VITE_API_BASE_URL.replace(/\/$/, '')
```
