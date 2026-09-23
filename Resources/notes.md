# 📝 Towel on the Sunbed — Notes & Learning Resources

> **TryHackMe — Hacker Holidays (Day 8)**
>
> **Room Category:** Web Security • Business Logic • Race Condition
>
> **Objective:** Understand how concurrent requests can bypass server-side reward logic when application state is not updated atomically.

---

# 🎯 Room Summary

**Towel on the Sunbed** demonstrates a **business logic race condition** in a cryptocurrency staking application. The application awards **50 PONZI** tokens every **24 hours** through a staking feature and unlocks a privileged **Whale Vault** after reaching **150 PONZI**.

Instead of waiting multiple days, the vulnerability is exploited by sending several authenticated reward requests **simultaneously**, causing the server to process multiple claims before updating the cooldown state.

> **Key Lesson:** Race conditions exploit timing, not malformed input.

---

# 🧠 Concepts Learned

## Business Logic Vulnerability

Business logic flaws occur when an application's workflow can be manipulated while still using legitimate functionality.

**Examples include:**

* Coupon reuse.
* Double spending.
* Reward duplication.
* Purchase workflow bypasses.
* Inventory race conditions.
* Loyalty point abuse.

This room focuses on **reward duplication** through concurrent execution.

---

## Race Condition

A race condition happens when multiple operations access and modify the same resource at nearly the same time.

### Expected Workflow

```text
Check eligibility
      │
      ▼
Grant reward
      │
      ▼
Update cooldown timestamp
```

### Vulnerable Workflow

```text
Request A ─┐
Request B ─┼── Check eligibility simultaneously
Request C ─┘
      │
      ▼
Multiple rewards granted
```

The application validates eligibility before the cooldown state is fully committed.

---

# 🛠️ Tools Used

| Tool                             | Purpose                               |
| -------------------------------- | ------------------------------------- |
| **TryHackMe**                    | Authorized lab environment.           |
| **Burp Suite Community Edition** | HTTP interception and request replay. |
| **Repeater**                     | Concurrent request execution.         |
| **Firefox**                      | Web interaction through proxy.        |

---

# 🌐 Enumeration Notes

Initial observations from the application:

* Cryptocurrency portfolio dashboard.
* User authentication required.
* Staking reward endpoint available.
* Whale Vault locked until reaching balance threshold.
* Cooldown timer visible after reward claim.

---

# 🔎 Important Endpoint

**Observed Endpoint**

```http
POST /claim
```

Purpose:

* Claims staking reward.
* Uses authenticated session.
* No request body required.
* Updates user balance.

**Authentication**

* Session maintained using an authenticated cookie.
* Browser session forwarded through Burp Suite.

---

# 📋 Testing Methodology

## Step 1 — Establish Baseline

* Register/login.
* Claim reward once.
* Observe successful balance update.
* Attempt another immediate request.

Result:

* Cooldown enforcement prevents sequential replay.

---

## Step 2 — Capture Request

Intercept reward request using Burp Suite Proxy.

Evidence collected:

* HTTP method.
* Endpoint.
* Headers.
* Session cookie.
* Origin and Referer.

---

## Step 3 — Replay Request

Send intercepted request into Burp Repeater.

Purpose:

* Replay identical authenticated request.
* Duplicate request multiple times.

---

## Step 4 — Parallel Request Testing

Create several identical requests.

Group requests inside Burp Repeater.

Execution mode:

> **Send Group in Parallel**

Purpose:

* Trigger simultaneous server-side processing.
* Test for timing issues.

---

## Step 5 — Validate State Change

Refresh dashboard after concurrent execution.

Observed indicators:

* Increased balance.
* Whale tier unlocked.
* Whale Vault becomes accessible.

---

# 📸 Screenshot Evidence

| Screenshot                      | Description                               |
| ------------------------------- | ----------------------------------------- |
| `01-dashboard-initial.png`      | Initial dashboard after login.            |
| `02-dashboard-staking.png`      | Reward panel and Whale Vault requirement. |
| `03-burp-captured-claim.png`    | Captured HTTP request.                    |
| `04-repeater-request.png`       | Request moved into Burp Repeater.         |
| `05-repeater-tab-group.png`     | Multiple requests grouped.                |
| `06-repeater-parallel-send.png` | Parallel execution option selected.       |
| `07-parallel-responses.png`     | Successful concurrent reward responses.   |
| `08-whale-vault-unlocked.png`   | Whale Vault unlocked after refresh.       |
| `09-flag-redacted.png`          | Completion evidence with flag hidden.     |
| `10-whale-vault-final.png`      | Final completed application state.        |

---

# 🔬 Root Cause Analysis

The reward operation is **not atomic**.

Possible vulnerable sequence:

```text
if eligible:
    grant_reward()
    update_cooldown()
```

When executed concurrently:

1. Multiple requests pass eligibility.
2. Each request grants reward.
3. Cooldown updated too late.
4. Balance increases multiple times.

---

# 🛡️ Defensive Recommendations

## Atomic Transactions

Reward validation and balance update should occur inside a single transaction.

## Row-Level Locking

Prevent simultaneous updates for the same user account.

## Idempotency Tokens

Ensure only one reward request is processed for a specific claim window.

## Server-Side Cooldown Enforcement

Never rely on client timing information.

## Concurrency Testing

Include automated tests for simultaneous requests during development.

---

# ⚠️ Security Impact

A similar vulnerability in production systems could allow:

* Duplicate reward claims.
* Loyalty point inflation.
* Cashback abuse.
* Cryptocurrency balance manipulation.
* Financial inconsistencies.

Race conditions often affect applications handling shared mutable state.

---

# 📚 Key Takeaways

* Race conditions exploit **timing** rather than input validation.
* Sequential testing alone may miss concurrency vulnerabilities.
* Burp Suite Repeater can simulate concurrent authenticated requests.
* Business logic testing is essential alongside traditional vulnerability testing.
* Proper synchronization is critical for financial and reward systems.

---

# 💡 MITRE ATT&CK Mapping (Educational)

| Technique                      | Relevance                                                  |
| ------------------------------ | ---------------------------------------------------------- |
| **Business Logic Abuse**       | Manipulating legitimate application functionality.         |
| **Valid Accounts**             | Exploitation performed through authenticated user actions. |
| **Application Layer Protocol** | HTTP requests used to trigger vulnerable workflow.         |

> This mapping is educational and intended for defensive understanding.

---

# 📖 Recommended Reading

### Web Security Topics

* Business Logic Vulnerabilities.
* Race Conditions in Web Applications.
* Idempotency in REST APIs.
* Transaction Isolation Levels.
* Secure Reward and Wallet Systems.

### Burp Suite Skills

* Proxy Interception.
* HTTP Repeater.
* Request Groups.
* Parallel Request Execution.
* Response Comparison.

---

# 🧾 Personal Learning Notes

**Skills practiced during this room**

* ✔ Business logic analysis.
* ✔ HTTP request interception.
* ✔ Session-aware testing.
* ✔ Concurrent request execution.
* ✔ Race condition validation.
* ✔ Security documentation and reporting.



**Primary Focus:** Business Logic / Race Condition

**Environment:** TryHackMe (Authorized Lab)

---

> **Portfolio Note:** This repository intentionally redacts the final flag while preserving the complete methodology, technical analysis, screenshots, and remediation guidance.
