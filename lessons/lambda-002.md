---
id: lambda-002
title: ES Modules Use .mjs Extension
services: [Lambda]
summary: esbuild with format 'esm' outputs .mjs files, not .js — package the correct extension or your zip is empty.
---

# ES Modules Use .mjs Extension

**Problem:** Deploy script couldn't find Lambda files - "zip warning: name not matched: lib/*.js"

**Root Cause:** esbuild with `format: 'esm'` outputs `.mjs` files, not `.js` files.

**Solution:** Package the correct extension:
```bash
zip -j auth.zip dist/auth/*.mjs  # NOT *.js
```
