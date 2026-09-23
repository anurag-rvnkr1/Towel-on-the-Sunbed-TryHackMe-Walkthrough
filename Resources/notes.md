# Resources / Notes

## Room Focus

**Towel on the Sunbed** is centered on a web-application business-logic race condition.

## Key Values

- Reward per successful claim: **50 PONZI**
- Whale Vault threshold: **150 PONZI**
- A cooldown is expected between claims.

## Workflow

```text
Browser
  ↓
Burp Proxy
  ↓
Capture POST /claim
  ↓
Send to Repeater
  ↓
Duplicate requests
  ↓
Create group
  ↓
Send group in parallel
  ↓
Compare responses
  ↓
Verify account state
```

## Core Observation

A single normal claim is allowed. A rapid sequential retry is rate-limited.

Concurrent requests can, however, overlap inside the state-check/update window and cause multiple reward grants.

## Documentation Rules

Do not publish:
- session cookies;
- authentication tokens;
- private infrastructure details;
- the challenge flag.

## Defensive Reference

For real applications, protect shared state using atomic transactions, proper locking/isolation, idempotency controls, server-side cooldown checks, database invariants, and concurrent regression tests.
