---
id: secrets-001
title: Never Pass Secrets as CloudFormation NoEcho Parameters
services: [Secrets Manager, CloudFormation, Lambda, EC2]
summary: NoEcho parameters can be clobbered (e.g. "UsePreviousValue" set literally) and aren't recoverable; store secrets in Secrets Manager and fetch ARNs at runtime.
---

# Never Pass Secrets as CloudFormation NoEcho Parameters

**Problem:** `aws cloudformation deploy --parameter-overrides JWTSecret=UsePreviousValue` set the literal string "UsePreviousValue" as the secret value (deploy doesn't support `UsePreviousValue=true` syntax). NoEcho parameters cannot be recovered once overwritten.

**Solution:** Store secrets in AWS Secrets Manager and have Lambdas/EC2 instances fetch them at runtime:
- Use a cached SecretsManager client that fetches once per Lambda cold start
- Lambda env vars contain `*_ARN` (e.g., `JWT_SECRET_ARN`) not secret values
- EC2 UserData uses `aws secretsmanager get-secret-value` at boot
- IAM roles need `secretsmanager:GetSecretValue` on `myapp-*` secrets

**Stack:** `myapp-secrets-dev` defines all secrets with cross-stack ARN exports.
