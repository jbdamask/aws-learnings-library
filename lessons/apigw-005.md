---
id: apigw-005
title: WebSocket API Gateway Has a 10-Minute Idle Connection Timeout
services: [API Gateway]
summary: WebSocket connections drop after 10 minutes of no messages — this is normal; reconnect or send keepalive pings.
---

# WebSocket API Gateway Has a 10-Minute Idle Connection Timeout

**Problem:** WebSocket connections appeared to "fail" repeatedly — browser showed reconnection cycles every ~10 minutes during long-running operations like server provisioning.

**Root Cause:** AWS API Gateway WebSocket has a default idle connection timeout of 10 minutes. If no messages are sent over the connection for 10 minutes, API Gateway closes it. The client then reconnects, which the user may perceive as "connection failures."

**Impact:** This is normal behavior, not a bug. During long operations with sparse updates, WebSocket connections will reconnect periodically. Messages sent during the brief reconnection window (~1-2 seconds) could be missed.

**Mitigation Options:**
- Accept periodic reconnections (simplest, acceptable for MVP)
- Implement server-side ping/pong frames every 5 minutes to keep connections alive
- On the client, queue/replay missed messages using a "last received timestamp" mechanism
