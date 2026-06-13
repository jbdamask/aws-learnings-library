---
id: eventbridge-002
title: Spot Interruption EventBridge Rule Must Target the Lambda ARN
services: [EventBridge, EC2, Lambda]
summary: An EventBridge rule for spot interruptions must target the Lambda ARN via !GetAtt Function.Arn, not the bare function name.
---

# Spot Interruption EventBridge Rule Must Target the Lambda ARN

**Problem:** A spot interruption EventBridge rule was configured but the handler Lambda never fired on interruption events.

**Root Cause:** EventBridge rule targets must reference the Lambda by its full ARN. Supplying just the function name (or otherwise not resolving the ARN) means the rule has no valid target and silently fails to invoke the handler.

**Solution:** Target the Lambda ARN explicitly using `!GetAtt`:
```yaml
SpotInterruptionRule:
  Type: AWS::Events::Rule
  Properties:
    EventPattern:
      source:
        - aws.ec2
      detail-type:
        - "EC2 Spot Instance Interruption Warning"
    Targets:
      - Arn: !GetAtt InterruptionHandlerFunction.Arn   # NOT the bare function name
        Id: SpotInterruptionHandler
```

**Key Insight:** When an EventBridge-triggered Lambda never runs, verify the rule target is the resolved ARN (`!GetAtt Function.Arn`) and that a corresponding `AWS::Lambda::Permission` grants `events.amazonaws.com` invoke rights.
