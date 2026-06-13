---
id: spot-001
title: Testing Spot Interruptions Without Waiting for Real Events
services: [EC2, Lambda, EventBridge]
summary: Real spot interruptions can't be triggered on demand and EventBridge blocks aws.* source PutEvents — invoke the handler Lambda directly with a synthetic payload.
---

# Testing Spot Interruptions Without Waiting for Real Events

**Problem:** Real EC2 spot interruptions are unpredictable and can't be triggered on demand, making it hard to test interruption handling.

**Solution:** Build a simulator endpoint that directly invokes the interruption handler Lambda with a synthetic EventBridge-style payload (using `Lambda InvokeCommand`), bypassing EventBridge itself. This is necessary because EventBridge blocks `PutEvents` with `aws.*` sources — only genuine AWS events can use those source prefixes.

**Verification Checklist:**
- [ ] Handler Lambda receives and processes the synthetic event
- [ ] Database records updated with interruption metadata
- [ ] Real-time notifications (WebSocket/push) delivered to the user
- [ ] Auto-replacement logic triggers if configured
- [ ] Original resource linked to its replacement in the database
