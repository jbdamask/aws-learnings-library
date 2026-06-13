---
id: lambda-004
title: Lambda Warm Containers Leak Module-Level State Across Invocations
services: [Lambda]
summary: Module-level objects persist across warm invocations; bind per-request identity (request IDs, log fields) inside the handler, never at module scope.
---

# Lambda Warm Containers Leak Module-Level State Across Invocations

**Problem:** Structured logs showed the same `run_id` (set from `context.aws_request_id`) across multiple Lambda invocations, making it impossible to tell how many times a job had actually been attempted. Per-invocation identity was silently stale.

**Root Cause:** Lambda reuses warm containers across invocations. Module-level objects — boto3 clients, loggers, engine instances — persist for the lifetime of the container. Any state bound at module initialization (including logger fields set via `.bind()`) carries over into subsequent invocations. If a logger or engine stores per-invocation identity at module scope, every subsequent warm invocation inherits the stale value from the first.

**Solution:** Module level is for clients and config only. Per-request identity must be set inside the handler function on every call:

```python
# Module level — correct: stateless clients and config only
_session = boto3.Session()
_llm = LlmClient(_session, ...)
_log = JsonLogger(component="my-worker")  # no per-request fields here

def lambda_handler(event, context):
    for record in event["Records"]:
        message = json.loads(record["body"])
        # Handler level — correct: bind per-invocation identity fresh every call
        log = _log.bind(
            run_id=context.aws_request_id,   # fresh from context, not cached
            job_id=message["job_id"],
        )
        engine = MyEngine(..., run_id=context.aws_request_id, log=log)
        engine.process(message)
```

**Key Insight:** Logger `.bind()` methods that mutate the parent object (rather than returning a new instance) are especially dangerous in Lambda — a single `bind()` call at module init or in a helper will pollute every subsequent invocation's logs. Always verify that `.bind()` returns a new object.
