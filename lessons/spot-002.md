---
id: spot-002
title: Spot Interruption Handler Needs the Same Env Vars as the Creation Lambda
services: [EC2, Lambda]
summary: If the interruption handler auto-replaces instances, it needs the same broad environment variables as your server-creation Lambda.
---

# Spot Interruption Handler Needs the Same Env Vars as the Creation Lambda

**Problem:** A spot interruption handler that performs auto-replacement failed to launch the replacement instance because it was missing configuration the creation path relies on.

**Root Cause:** Auto-replacement re-runs the same server-creation logic (launch templates, AMI IDs, subnet/security-group IDs, secret ARNs, callback URLs, etc.). If the interruption handler Lambda is provisioned with a minimal env-var set, it lacks the configuration the creation Lambda has, and replacement silently fails or launches a misconfigured instance.

**Solution:** When the interruption handler does auto-replacement, give it the **same broad set of environment variables** as your server-creation Lambda. Share the configuration via a common CloudFormation mapping/parameter set so the two Lambdas can't drift apart.

**Key Insight:** Any Lambda that re-creates infrastructure needs the full creation configuration, not just interruption metadata. Treat the interruption handler as a second entry point into the creation path.
