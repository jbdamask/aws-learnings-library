---
id: eventbridge-001
title: EventBridge PutEvents Can Return 200 OK With Failed Entries
services: [EventBridge]
summary: PutEvents returns HTTP 200 even when individual entries fail; always check FailedEntryCount in the response body.
---

# EventBridge PutEvents Can Return 200 OK With Failed Entries

**Problem:** Admin endpoint reported "event injected successfully" but the EventBridge handler Lambda was never triggered.

**Root Cause:** `EventBridgeClient.send(PutEventsCommand)` returns HTTP 200 even when individual entries fail to inject. Failures are reported in the response body via `FailedEntryCount` and per-entry `ErrorCode`/`ErrorMessage` fields.

**Solution:** Always check `FailedEntryCount` after PutEvents:
```typescript
const result = await eventBridge.send(new PutEventsCommand({ Entries: [...] }))
if (result.FailedEntryCount && result.FailedEntryCount > 0) {
  console.error('EventBridge PutEvents had failures:', JSON.stringify(result.Entries))
  throw new Error('Failed to inject event into EventBridge')
}
```
