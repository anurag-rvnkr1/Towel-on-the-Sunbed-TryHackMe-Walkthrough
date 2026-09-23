---
layout: default
title: "Towel on the Sunbed | TryHackMe"
description: "Portfolio documentation for the Towel on the Sunbed TryHackMe challenge."
---

# 🏖️ Towel on the Sunbed

## TryHackMe CTF Walkthrough

> **Web Security • Business Logic • Race Condition • Burp Suite**

This portfolio page documents an authorized TryHackMe challenge in which a cryptocurrency staking workflow can be manipulated through concurrent requests.

## Challenge Profile

| Field | Details |
|---|---|
| Platform | TryHackMe |
| Challenge | Towel on the Sunbed |
| Category | Web / Business Logic |
| Core issue | Race condition |
| Primary tooling | Burp Suite Repeater |
| Evidence | 10 numbered screenshots |
| Flag | **Redacted** |

## Assessment Flow

```text
01  Application review
02  Reward endpoint capture
03  Sequential baseline
04  Repeater preparation
05  Request grouping
06  Parallel dispatch
07  Response comparison
08  State verification
09  Vault evidence
10  Final-state documentation
```

## 01 — Application Review

The dashboard exposes the staking reward workflow and Whale Vault progression.

![Initial dashboard](assets/01-dashboard-initial.png)

![Staking workflow](assets/02-dashboard-staking.png)

## 02 — HTTP Interception

The reward operation was captured with Burp Suite Proxy.

![Captured request](assets/03-burp-captured-claim.png)

## 03 — Repeater Preparation

The request was transferred to Repeater and duplicated for controlled concurrency testing.

![Repeater request](assets/04-repeater-request.png)

![Request group](assets/05-repeater-tab-group.png)

## 04 — Parallel Race Test

The grouped requests were sent in parallel and the responses compared.

![Parallel execution](assets/06-repeater-parallel-send.png)

![Parallel responses](assets/07-parallel-responses.png)

The key finding was that concurrent execution allowed multiple reward operations to succeed inside the same timing window.

## 05 — Application Verification

The dashboard was refreshed after the concurrency test.

![Vault unlocked](assets/08-whale-vault-unlocked.png)

The Whale Vault became available after the balance crossed the configured threshold.

## 06 — Final Evidence

The final vault evidence is included for completeness, but the flag itself is not published.

![Redacted flag evidence](assets/09-flag-redacted.png)

![Final state](assets/10-whale-vault-final.png)

## Vulnerability Explanation

The issue can be modelled as:

```text
                 ┌──────────────┐
                 │  POST /claim │
                 └──────┬───────┘
                        │
                  eligibility check
                        │
                ┌───────┴────────┐
                │ concurrent race │
                └───────┬────────┘
                        │
                multiple updates
                        │
                    balance gain
```

A secure design should make eligibility validation and balance mutation one atomic state transition.

## Remediation

Production systems should consider:

- transactional/atomic balance updates;
- appropriate locks or isolation;
- idempotency keys;
- server-side cooldown enforcement;
- database constraints;
- concurrent request regression tests.

## Documentation

For the complete narrative report, see [`Documentation/Documentation.md`](../Documentation/Documentation.md).

## Public Flag Policy

The challenge flag is intentionally redacted to reduce spoilers, plagiarism, and simple answer copying. The repository focuses on the investigation process, technical evidence, and security lessons.

## Responsible Use

Use these techniques only against systems where you have explicit authorization.

---

**Author:** Anurag  
**Repository:** `Towel-on-the-Sunbed-TryHackMe-Walkthrough`
