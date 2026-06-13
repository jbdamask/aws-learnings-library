---
id: cloudfront-002
title: CloudFront Signed-Cookie Key Rotation Must Use Two-Key Overlap
services: [CloudFront, Secrets Manager, SSM]
summary: Hard-cutover key rotation (add new + delete old in one step) 403s every active signed cookie for ~1h; keep both keys in the KeyGroup and sweep old keys after a grace window.
---

# CloudFront Signed-Cookie Key Rotation Must Use Two-Key Overlap

**Problem:** Rotated the CloudFront signing key by adding the new public key to the KeyGroup, swapping Secrets Manager + SSM to the new key, then immediately removing the old key from the KeyGroup AND deleting the `PublicKey` resource. Every authenticated user got `403 InvalidKey: Unknown Key` on private content for ~1 hour after each rotation.

**Root Cause — two compounding problems:**

1. **Signed-cookie TTL.** CloudFront signed cookies are valid for hours-to-days. Browsers don't proactively re-fetch cookies the moment the key rotates — they keep sending the cookie they already have. The instant the old key is removed/deleted, every existing cookie is rejected.
2. **Auth-Lambda module-level cache.** Auth Lambdas typically cache the loaded private key + Key-Pair-Id at module scope for warm-container reuse. After a rotation, warm containers continue **issuing fresh cookies signed with the now-deleted key** until the container cycles (~5–60 min idle). So even users who proactively refresh keep getting bad cookies until both their browser refreshes AND the auth Lambda has cycled to a cold start.

**AWS official guidance** ([Specify signers that can create signed URLs and signed cookies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-trusted-signers.html)):

> To rotate key pairs that you're using to create signed URLs or signed cookies without invalidating URLs or cookies that haven't expired yet, create a new key pair and add the public key to a key group. **Don't remove any existing public keys from the key group yet — only add the new ones.** Only after existing signed URLs and cookies have expired should you remove the old keys from the key group.

KeyGroups support up to 5 keys precisely to enable this overlap pattern.

**Solution — two-key overlap with deferred sweep:**

```python
def _rotate_key():
    # 1-4. Generate, register, add to KG, verify post-add (unchanged)
    new_key_pair_id = _register_public_key(public_pem)
    _add_to_key_group(new_key_pair_id)
    _verify_key_in_group(new_key_pair_id, expected_present=True)

    # 5-6. Update secret + SSM (unchanged)
    _store_secret(private_pem, new_key_pair_id)
    ssm.put_parameter(Name=KEY_PAIR_ID_PARAM, Value=new_key_pair_id, ...)

    # 7. SWEEP keys older than grace_window — NOT just the immediately-prior key
    _sweep_expired_keys(grace_window_hours=24, current_id=new_key_pair_id)

    # 8. Emit success metric, then log "Rotated:"

def _sweep_expired_keys(grace_window_hours, current_id):
    """Remove KeyGroup members and delete PublicKeys older than the grace
    window. Never touches the current key. Idempotent — safe to call on
    every rotation. The first rotation after this fix lands will leave the
    pre-fix old key untouched (still within its grace window); the next
    rotation will sweep it out."""
    cutoff = datetime.utcnow() - timedelta(hours=grace_window_hours)
    for pk in cf.list_public_keys()["PublicKeyList"]["Items"]:
        if pk["Id"] == current_id:
            continue
        if pk["CreatedTime"] >= cutoff:
            continue  # still within grace window
        _remove_from_key_group(pk["Id"])  # idempotent — no-op if already gone
        cf.delete_public_key(Id=pk["Id"], IfMatch=resp["ETag"])
```

**Grace window sizing:** must exceed both (a) the longest cookie expiry your app actually issues AND (b) the auth-Lambda warm-container TTL with margin. For most setups 24h is the floor; 48h gives margin for users who close their laptop overnight.

**Key Insight:** "Working in test" doesn't catch this — test usually has 1 user, so a single sign-out / sign-in cycle masks the disruption. The bug only manifests at scale, every rotation cycle. If your rotation Lambda removes the old key in the same invocation that adds the new one, you have this bug.

**Bonus gotcha:** auth-Lambda module-level caches of cryptographic material need an invalidation strategy if you ever want fast key turnover. Either (a) periodically refresh the cache (e.g., re-read secret every 5 min), (b) use a TTL on the cached values, or (c) accept the warm-container TTL as a hard floor on rotation propagation.

**Verification before re-enabling rotation schedules:**
- Trace a code path where `_rotate_key` adds a new key but does NOT remove the prior key in the same invocation.
- Confirm the sweeper exists and is gated by both age AND not-currently-active.
- Test: simulate a rotation, verify both keys are in the KeyGroup after rotation, verify a cookie signed with the prior key still validates, then run a second rotation and verify the prior key gets swept.
