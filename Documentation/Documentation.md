# 🌞 Towel on the Sunbed — TryHackMe Walkthrough

<div align="center">

![TryHackMe](https://img.shields.io/badge/TryHackMe-Hacker%20Holidays-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-success?style=for-the-badge)
![Category](https://img.shields.io/badge/Web-Business%20Logic-blue?style=for-the-badge)
![Vulnerability](https://img.shields.io/badge/Vulnerability-Race%20Condition-orange?style=for-the-badge)
![Burp Suite](https://img.shields.io/badge/Burp%20Suite-Repeater-F26C21?style=for-the-badge&logo=burpsuite)

# Professional CTF Documentation

**Hacker Holidays — Day 8**

*Business Logic Race Condition Exploitation using Burp Suite Repeater*

**Author:** Anurag Revankar

Cybersecurity Portfolio Documentation

</div>

---

# Table of Contents

1. Executive Summary
2. Room Overview
3. Learning Objectives
4. Skills Demonstrated
5. Lab Environment
6. Vulnerability Overview
7. Attack Surface Analysis
8. Step 1 — Application Enumeration
9. Step 2 — Understanding the Reward Workflow
10. Step 3 — Capturing the Request
11. Technical Notes
12. Key Takeaways

---

# Executive Summary

**Towel on the Sunbed** is a web application challenge from the **TryHackMe Hacker Holidays** series that focuses on identifying and exploiting a **business logic race condition** within a cryptocurrency staking platform.

Unlike traditional web exploitation challenges involving SQL Injection, XSS, SSTI, or Remote Code Execution, this room demonstrates how legitimate application functionality can become vulnerable when server-side state changes are processed concurrently.

The application implements a staking system that awards users **50 PONZI tokens** every **24 hours**. The Whale Vault becomes available only after accumulating **150 PONZI**, implying users should normally wait multiple days before accessing the vault.

During this assessment, authenticated reward requests were intercepted using **Burp Suite**, duplicated in **Repeater**, grouped together, and dispatched simultaneously. Because the application validated multiple requests before updating the cooldown timestamp, several rewards were processed successfully during a single execution window.

This documentation explains the complete methodology, evidence collection process, technical analysis, security impact, and remediation recommendations while intentionally **redacting the final challenge flag** for responsible public documentation.

---

# Room Overview

| Property | Details |
|----------|---------|
| **Platform** | TryHackMe |
| **Room** | Towel on the Sunbed |
| **Series** | Hacker Holidays |
| **Challenge Day** | Day 8 |
| **Difficulty** | Easy |
| **Category** | Web Application Security |
| **Primary Vulnerability** | Business Logic Race Condition |
| **Tools Used** | Burp Suite Community Edition, Firefox |
| **Operating Environment** | Linux Desktop |

---

# Learning Objectives

The objectives completed during this room include:

- Enumerate a web application's functionality.
- Understand server-side reward workflows.
- Identify business logic assumptions.
- Capture authenticated HTTP requests.
- Observe cooldown enforcement.
- Perform concurrent request testing.
- Trigger multiple reward claims.
- Validate changes within application state.
- Document findings professionally.

---

# Skills Demonstrated

## Web Application Security

- HTTP request interception.
- Session-aware request replay.
- Authentication handling.
- Application workflow analysis.

## Burp Suite

- Proxy configuration.
- Request interception.
- Repeater usage.
- Request duplication.
- Parallel request groups.

## Business Logic Testing

- Workflow abuse.
- Race condition testing.
- State transition analysis.
- Cooldown bypass validation.

## Reporting

- Screenshot evidence collection.
- Technical documentation.
- Security impact assessment.
- Defensive recommendations.

---

# Lab Environment

The challenge simulates a cryptocurrency investment dashboard.

## Components Observed

| Component | Purpose |
|-----------|---------|
| Portfolio Dashboard | Displays user balance. |
| Market Prices | Cosmetic cryptocurrency prices. |
| Staking Rewards | Daily reward mechanism. |
| Whale Vault | Reward unlocked after reaching threshold. |
| Authentication | Session-based login. |

## Tools Configuration

| Tool | Configuration |
|------|---------------|
| Firefox | Configured through Burp proxy. |
| Burp Suite | Intercept enabled for HTTP traffic. |
| Repeater | Used for concurrent request replay. |

---

# Vulnerability Overview

## What is a Business Logic Vulnerability?

Business logic vulnerabilities occur when an attacker abuses the application's intended workflow instead of exploiting malformed input.

Examples include:

- Reward duplication.
- Coupon reuse.
- Inventory abuse.
- Purchase manipulation.
- Double spending.
- Race conditions.

The application behaves exactly as designed during ordinary sequential use, but fails when legitimate actions occur simultaneously.

---

# Understanding Race Conditions

A race condition appears when two or more requests interact with shared application state before updates are completed.

## Expected Secure Workflow

```text
User clicks Claim Reward
        │
        ▼
Server validates cooldown
        │
        ▼
Reward granted once
        │
        ▼
Cooldown timestamp updated
```

Only one reward should ever be processed.

---

## Vulnerable Workflow

```text
 Request A
     │
 Request B
     │
 Request C
     │
     ▼
Eligibility validated simultaneously
     ▼
Multiple reward updates committed
     ▼
Cooldown updated too late
```

The application's validation and update operations are not performed atomically.

---

# Attack Surface Analysis

Before exploitation, the visible application components were reviewed.

## Authentication

- Session cookie generated after login.
- Authenticated dashboard.
- Reward endpoint accessible only after authentication.

## Reward Functionality

The dashboard exposes one important feature:

> **Claim Reward**

Characteristics:

- Daily staking reward.
- Fixed reward amount.
- Cooldown enforced after successful claim.

## Whale Vault

The Whale Vault remains inaccessible until balance reaches **150 PONZI**.

This immediately suggests that reward accumulation controls access to privileged functionality.

---

# Step 1 — Initial Application Enumeration

After logging into the application, the dashboard presents the initial cryptocurrency portfolio.

![Initial Dashboard](../docs/assets/01-dashboard-initial.png)

**Figure 1 — Initial Portfolio Dashboard**

### Observations

The dashboard contains:

- Portfolio balance.
- User tier.
- Cryptocurrency prices.
- Daily staking reward.
- Whale Vault progression.

No sensitive functionality is immediately exposed beyond the reward mechanism.

---

## Enumeration Notes

| Observation | Notes |
|-------------|------|
| Balance | Initially zero tokens. |
| Reward Button | Immediately available. |
| Whale Vault | Locked until threshold reached. |
| Market Prices | Informational only. |

The reward workflow becomes the primary target for testing.

---

# Step 2 — Understanding the Reward Workflow

Scrolling further reveals the staking component.

![Reward Mechanism](../docs/assets/02-dashboard-staking.png)

**Figure 2 — Staking Reward Panel**

### Functional Analysis

The application states:

- Earn **50 PONZI** every **24 hours**.
- Reward available now.
- Whale Vault unlocks at **150 PONZI**.

### Expected User Journey

| Day | Expected Balance |
|-----|------------------|
| Day 1 | 50 |
| Day 2 | 100 |
| Day 3 | 150 |
| Vault | Accessible |

The intended progression clearly requires waiting multiple days.

---

## Business Logic Assumption

The application assumes users cannot claim rewards concurrently.

This assumption becomes the target of testing.

---

# Threat Modeling

### Valuable Asset

- Cryptocurrency balance.

### Security Control

- 24-hour cooldown.

### Trust Boundary

- Server validates reward eligibility.

### Potential Weakness

- Concurrent requests reaching validation simultaneously.

---

# Step 3 — Capturing the Reward Request

The browser is configured to send traffic through Burp Suite.

Proxy interception is enabled before clicking **Claim Reward**.

![Burp Interception](../docs/assets/03-burp-captured-claim.png)

**Figure 3 — Intercepted Reward Request**

The request reveals a simple authenticated endpoint responsible for claiming staking rewards.

### Request Characteristics

| Property | Observation |
|----------|-------------|
| Method | POST |
| Endpoint | `/claim` |
| Authentication | Session cookie |
| Request Body | Empty |
| Action | State-changing request |

### Why This Endpoint Matters

This endpoint directly modifies the authenticated user's balance.

State-changing endpoints are excellent candidates for:

- Replay testing.
- Race condition testing.
- Idempotency testing.
- Authorization validation.

---

# HTTP Request Analysis

The captured request contains:

```http
POST /claim HTTP/1.1
Host: <LAB_HOST>

Cookie: connect.sid=<SESSION>

Origin: <LAB_ORIGIN>

Referer: <LAB_REFERER>
```

### Sensitive Information Redacted

For responsible public documentation:

- Session cookie removed.
- Lab host generalized.
- User identifiers omitted.

This preserves methodology without exposing reusable credentials.

---

# Baseline Behaviour Testing

The intercepted request is forwarded normally.

### Expected Response

The first request succeeds.

The reward is credited.

### Immediate Replay

A second sequential request is attempted.

Result:

- Cooldown enforced.
- Reward rejected.

This confirms the application performs cooldown validation during ordinary sequential execution.

---

# Security Observation

The endpoint is protected against **simple replay attacks**.

However, sequential replay protection alone does **not** guarantee protection against concurrent execution.

This distinction becomes the focus of the next stage.

---


---

# Step 4 — Moving the Request into Burp Suite Repeater

After confirming that sequential reward claims were blocked by the application's cooldown mechanism, the next objective was to determine whether the same protection could be bypassed under **concurrent execution**.

Rather than modifying request parameters or authentication data, the exact authenticated request captured earlier was transferred into **Burp Suite Repeater** for controlled testing.

![Repeater Request](../docs/assets/04-repeater-request.png)

**Figure 4 — Authenticated reward request inside Burp Suite Repeater**

---

## Why Use Burp Repeater?

Burp Repeater is designed for replaying HTTP requests while preserving authentication, headers, cookies, and request structure.

For race condition testing it provides several advantages:

| Feature | Why it Matters |
|---------|----------------|
| Replay authenticated requests | Uses the current session without re-authentication. |
| Duplicate identical requests | Creates multiple concurrent requests easily. |
| Group requests together | Enables synchronized execution. |
| Parallel send | Simulates simultaneous client requests. |
| Compare responses | Helps identify inconsistent server behavior. |

The important principle during this phase was:

> **No payload modification. No header manipulation. No authentication bypass.**

The exploit relies entirely on **timing**.

---

# Understanding Sequential vs Parallel Execution

Before sending concurrent requests, it is useful to understand why sequential replay behaves differently.

## Sequential Execution

```text
Request #1
    │
    ▼
Eligibility Check
    ▼
Reward Granted
    ▼
Cooldown Updated
    ▼
Request #2
    ▼
Cooldown Check
    ▼
Rejected
```

Only one request succeeds.

---

## Parallel Execution

```text
Request A ──────────────┐
                        │
Request B ──────────────┼────► Server receives requests together
                        │
Request C ──────────────┘

             ▼
Multiple eligibility checks occur before
shared account state is updated.
```

This timing difference is the foundation of the vulnerability.

---

# Creating Multiple Identical Requests

The captured request was duplicated several times.

### Methodology

1. Keep the original intercepted request.
2. Send it to Repeater.
3. Duplicate the tab multiple times.
4. Ensure every request is identical.
5. Preserve authentication cookie.
6. Preserve HTTP method and endpoint.

### Validation Checklist

| Item | Status |
|------|--------|
| Same HTTP Method | ✅ |
| Same Endpoint | ✅ |
| Same Cookie | ✅ |
| Same Headers | ✅ |
| Same Body | ✅ |

This guarantees that timing—not request content—is being tested.

---

# Step 5 — Creating a Request Group

The duplicated requests were added into a **Repeater Group**.

![Repeater Group](../docs/assets/05-repeater-tab-group.png)

**Figure 5 — Multiple identical reward requests grouped inside Burp Repeater**

Grouping requests allows Burp Suite to coordinate execution instead of sending requests one after another.

---

## Why Group Requests?

Without grouping:

```text
Request 1
Request 2
Request 3
```

Each request waits for the previous one.

With grouping:

```text
Request 1
Request 2
Request 3
```

All requests leave Burp at nearly the same moment.

---

## Race Window

A race window is the short period during which shared application state has not yet been updated.

Example timeline:

```text
Time
│
├── Request A reaches server
├── Request B reaches server
├── Request C reaches server
│
├── Eligibility still TRUE
│
├── Reward Processing
├── Reward Processing
├── Reward Processing
│
└── Cooldown Updated (too late)
```

Every request observes the same valid state.

---

# Step 6 — Executing Requests in Parallel

Burp Suite provides **Send Group in Parallel**.

![Parallel Execution](../docs/assets/06-repeater-parallel-send.png)

**Figure 6 — Sending grouped reward requests simultaneously**

### Execution Process

1. Select request group.
2. Choose **Parallel**.
3. Send requests.
4. Observe responses.

No manual delay is introduced.

The requests are transmitted concurrently.

---

## Why Parallel Execution Matters

The vulnerability exists because multiple server threads/processes begin processing before the cooldown timestamp changes.

Conceptually:

```text
Thread A
Thread B
Thread C

▼

Shared Reward State

Balance = 0
Cooldown = Available
```

Every thread begins with identical state.

---

# Concurrency Testing Methodology

### Objective

Determine whether reward processing is **atomic**.

### Test Conditions

| Condition | Value |
|-----------|-------|
| Authentication | Same session |
| Request Body | Identical |
| Endpoint | Same |
| Timing | Concurrent |
| Payload Mutation | None |

### Success Criteria

More than one request receives a successful reward response.

---

# Request Timeline Analysis

## Expected Server Behavior

```text
Request A
    ▼
Lock Account
    ▼
Grant Reward
    ▼
Update Cooldown
    ▼
Unlock Account

Request B
    ▼
Cooldown Active
    ▼
Reject
```

---

## Observed Behavior

```text
Request A
Request B
Request C

        ▼
Eligibility evaluated concurrently

        ▼
Reward Granted
Reward Granted
Reward Granted

        ▼
Cooldown Updated
```

The cooldown was enforced **after** rewards had already been processed.

---

# Step 7 — Response Analysis

The responses returned from Burp Repeater demonstrated inconsistent application behavior.

![Parallel Responses](../docs/assets/07-parallel-responses.png)

**Figure 7 — Multiple successful responses from concurrent reward requests**

---

## Response Comparison

| Request | Response |
|---------|----------|
| Request A | Success |
| Request B | Success |
| Request C | Success |

Instead of rejecting duplicate requests, multiple requests returned successful reward messages.

---

## Why This Is Significant

Sequential replay had already shown cooldown protection.

Parallel replay bypassed that protection.

This indicates the application is vulnerable specifically to **concurrent state transitions**.

---

# Response Consistency Review

The successful responses contained:

- Successful reward confirmation.
- Updated balance.
- Reward amount.
- HTTP success status.

The important observation is that multiple requests independently believed they were eligible.

---

# Balance Progression Analysis

### Intended Progression

| Action | Balance |
|--------|---------|
| Initial | 0 |
| Claim Once | 50 |
| Cooldown | 50 |

### Observed Progression

| Action | Balance |
|--------|---------|
| Initial | 0 |
| Parallel Claim | 150 |
| Refresh | Vault Available |

The balance exceeded the intended limit immediately.

---

# Why Cooldown Failed

The cooldown check was likely separated from the balance update.

Possible vulnerable sequence:

```python
if reward_available(user):
    add_reward(user)
    save_cooldown(user)
```

If several requests evaluate `reward_available()` before `save_cooldown()` completes, every request succeeds.

---

# Atomicity Explained

## Secure Transaction

```text
START TRANSACTION

Check cooldown
Update balance
Update cooldown

COMMIT
```

All operations succeed together.

---

## Vulnerable Transaction

```text
Check cooldown

Request Interrupted

Second request checks cooldown

Both continue
```

Shared state becomes inconsistent.

---

# State Transition Visualization

```text
Shared State

Balance = 0
Cooldown = False

        │
────────┼───────────────
        │

Request A checks cooldown ✔
Request B checks cooldown ✔
Request C checks cooldown ✔

        │
────────┼───────────────

Reward Added (+50)
Reward Added (+50)
Reward Added (+50)

        │
────────┼───────────────

Cooldown Updated
```

Three successful rewards occur during one cooldown period.

---

# Root Cause Discussion

The vulnerability is not caused by:

- SQL Injection
- XSS
- Authentication bypass
- Session fixation
- CSRF

Instead it results from:

- Shared mutable state.
- Missing synchronization.
- Missing transaction isolation.
- Non-idempotent reward processing.

---

# Race Condition Characteristics Observed

| Property | Observation |
|----------|-------------|
| Authentication Required | Yes |
| Input Manipulation Needed | No |
| Multiple Requests Needed | Yes |
| Timing Sensitive | Yes |
| State-Changing Endpoint | Yes |

---

# HTTP-Level Indicators

During testing, several useful indicators were collected.

### Successful Indicators

- HTTP Success responses.
- Updated reward balances.
- Multiple accepted operations.

### Defensive Indicator

Sequential requests produced cooldown enforcement, confirming business logic existed but was incomplete.

---

# Why This Is a Business Logic Issue

The application trusts a workflow assumption:

> One user submits one reward request at a time.

The attacker does not violate authentication or authorization.

Instead, legitimate functionality is abused through unexpected timing.

---

# Secure Design Principle Violated

## One-Time Action

The reward endpoint should be:

- Atomic.
- Idempotent.
- Serialized.

Instead it behaves as:

- Concurrent.
- Repeatable.
- Non-atomic.

---

# Evidence Summary (Exploitation Phase)

| Figure | Evidence |
|--------|----------|
| Figure 4 | Captured request moved into Repeater. |
| Figure 5 | Requests grouped for synchronized execution. |
| Figure 6 | Parallel send initiated. |
| Figure 7 | Multiple successful reward responses returned. |

These screenshots establish the complete exploitation chain without exposing challenge secrets.

---

### Tools Practiced

- Burp Proxy.
- Burp Repeater.
- Parallel Request Groups.
- Response comparison.

---

---

# Step 8 — Verifying the Whale Vault Unlock

After executing the grouped requests in parallel, Burp Suite interception was disabled so the browser could communicate with the application normally.

The dashboard was refreshed to verify whether the application's internal state had actually changed.

![Whale Vault Unlocked](../docs/assets/08-whale-vault-unlocked.png)

**Figure 8 — Whale Vault becomes accessible after concurrent reward processing**

---

## Validation Objective

Receiving successful HTTP responses alone is not enough to prove exploitation.

A second validation step is necessary to confirm that:

- the balance stored on the server has changed,
- privilege progression has updated,
- protected functionality is now accessible.

Refreshing the application confirmed all three conditions.

---

## Application State Before Refresh

| Property | Value |
|----------|------|
| Balance | Initial reward balance |
| Whale Vault | Locked |
| Whale Status | Not Eligible |

---

## Application State After Refresh

| Property | Value |
|----------|------|
| Balance | Increased beyond threshold |
| Whale Vault | **Unlocked** |
| Whale Status | Eligible |

This confirms the race condition modified persistent application state rather than producing temporary client-side behavior.

---

# Step 9 — Whale Vault Access

With the required balance reached, the **Open Vault** action became available.

Selecting the vault completed the challenge workflow.

---

## Completion Evidence

For portfolio purposes, completion evidence is documented without exposing the challenge solution.

![Redacted Flag](../docs/assets/09-flag-redacted.png)

**Figure 9 — Final challenge evidence with the flag intentionally hidden**

---

## Why the Flag Is Hidden

This repository is intended for:

- cybersecurity portfolio presentation,
- documentation quality,
- technical learning.

Publishing the flag would reduce the educational value of the room and create plagiarism issues.

The screenshot demonstrates successful completion while masking the sensitive value.

---

# Final Application State

The dashboard reflects the completed challenge after exploiting the race condition.

![Final State](../docs/assets/10-whale-vault-final.png)

**Figure 10 — Final application state after successful exploitation**

The challenge objective was successfully achieved through manipulation of application timing rather than unauthorized authentication or input injection.

---

# Attack Flow Summary

The complete exploitation chain can be represented as follows.

```text
                 User Authentication
                        │
                        ▼
          Cryptocurrency Portfolio Dashboard
                        │
                        ▼
             Review Reward Mechanism
                        │
                        ▼
           Intercept Reward HTTP Request
                        │
                        ▼
            Establish Baseline Behaviour
                        │
                        ▼
         Duplicate Authenticated Requests
                        │
                        ▼
         Create Burp Repeater Request Group
                        │
                        ▼
         Send Requests in Parallel
                        │
                        ▼
      Multiple Rewards Credited Concurrently
                        │
                        ▼
           Whale Vault Unlocks Successfully
                        │
                        ▼
        Capture Completion Evidence (Redacted)
```

---

# Technical Root Cause Analysis

## Business Logic Review

The application's reward functionality depends on a cooldown mechanism.

The expected design is:

```text
Validate Eligibility
        │
        ▼
Grant Reward
        │
        ▼
Update Cooldown Timestamp
        │
        ▼
Reject Future Requests
```

This assumes requests are processed sequentially.

---

## Vulnerable Processing Model

The observed behavior suggests the application processes requests similarly to:

```python
if cooldown_expired(user):
    add_reward(user, 50)
    update_last_claim(user)
```

When several requests execute simultaneously:

1. Request A validates cooldown.
2. Request B validates cooldown.
3. Request C validates cooldown.
4. Reward added multiple times.
5. Cooldown updated afterwards.

The validation step is not synchronized with the state update.

---

## Missing Atomic State Transition

A secure implementation should treat reward processing as one indivisible transaction.

### Secure Workflow

```text
BEGIN TRANSACTION

Check cooldown.
Lock reward record.
Update balance.
Record timestamp.

COMMIT
```

Only one request succeeds.

### Vulnerable Workflow

```text
Check cooldown.
Check cooldown.
Check cooldown.

Reward update.
Reward update.
Reward update.

Cooldown saved afterwards.
```

Multiple requests observe stale state simultaneously.

---

# Why Sequential Replay Did Not Work

Earlier testing showed immediate sequential requests returned cooldown enforcement.

That protection exists because:

- first request completes,
- timestamp is stored,
- second request sees updated state.

Parallel execution changes this timeline.

---

# Race Window Explained

A race window is the short interval between **validation** and **commit**.

```text
Eligibility Check
        │
        │  ← Race Window
        │
Balance Update
        │
Cooldown Update
```

Every concurrent request entered during this window.

---

# Security Impact Assessment

## Potential Real-World Impact

A similar flaw inside a production application could affect:

| System | Potential Impact |
|--------|------------------|
| Cryptocurrency Wallet | Duplicate rewards or balances. |
| Banking Application | Double credit transactions. |
| Cashback Platform | Cashback duplication. |
| Loyalty Program | Unlimited reward farming. |
| E-commerce Coupons | Multiple coupon redemption. |
| Ticket Reservation | Double booking inventory. |

---

## Business Impact

Possible consequences include:

- Financial loss.
- Reward inflation.
- Abuse of promotional systems.
- Account privilege escalation through rewards.
- Integrity violations in transaction history.

---

## CIA Impact Analysis

| Principle | Impact |
|-----------|--------|
| **Confidentiality** | Low |
| **Integrity** | **High** |
| **Availability** | Low |

The vulnerability primarily affects **data integrity**.

---

# Vulnerability Classification

| Classification | Mapping |
|---------------|---------|
| Vulnerability Type | Race Condition |
| Security Domain | Business Logic |
| Attack Vector | Authenticated HTTP Requests |
| Authentication Required | Yes |
| Privilege Required | Low |
| User Interaction | Required |
| Exploit Complexity | Low |

---

# MITRE ATT&CK Mapping (Educational)

| ATT&CK Technique | Relevance |
|------------------|----------|
| **Valid Accounts (T1078)** | Exploit uses authenticated session. |
| **Application Layer Protocol (T1071)** | HTTP used for exploitation. |
| **Exploitation for Privilege Escalation** | Business functionality unlocked. |
| **Exploitation of Public-Facing Application** | Vulnerable web endpoint tested. |

> This mapping is educational rather than an assertion that the room represents a real ATT&CK scenario.

---

# OWASP Top 10 Mapping

| OWASP Category | Relevance |
|----------------|----------|
| **A01 — Broken Access Control** | Privileged functionality unlocked through logic flaw. |
| **A04 — Insecure Design** | Reward workflow lacks concurrency protection. |
| **A05 — Security Misconfiguration** | Missing synchronization around state changes. |

The primary alignment is **Insecure Design** because the workflow itself is flawed.

---

# Blue Team Detection Opportunities

A SOC analyst could identify this behavior through application telemetry.

## Indicators of Suspicious Activity

- Multiple reward requests within milliseconds.
- Identical authenticated requests.
- Same session generating simultaneous POST requests.
- Balance increasing unusually quickly.

---

## Useful Log Fields

| Field | Detection Value |
|-------|-----------------|
| Session ID | Same session across requests. |
| User ID | Multiple reward events. |
| Timestamp | Near-identical timestamps. |
| Endpoint | `/claim` |
| Response Status | Multiple success responses. |

---

## Example Detection Logic

```text
IF

POST /claim

appears multiple times

within one second

for the same authenticated session

THEN

Generate business logic race-condition alert.
```

---

# Secure Development Recommendations

## 1. Atomic Transactions

Reward updates should occur inside a single transaction.

## 2. Row-Level Locking

Prevent concurrent updates to the same account balance.

## 3. Optimistic Locking

Reject conflicting concurrent updates.

## 4. Idempotency Tokens

Process each reward request only once.

## 5. Cooldown Validation

Validate cooldown immediately before committing state.

## 6. Server-Side State Enforcement

Do not trust client timing or UI timers.

---

# Secure Architecture Example

```text
User
 │
 ▼
API Gateway
 │
 ▼
Reward Service
 │
 ▼
Transaction Lock
 │
 ▼
Eligibility Check
 │
 ▼
Balance Update
 │
 ▼
Cooldown Update
 │
 ▼
Audit Log
 │
 ▼
Response
```

This architecture removes the race window.

---

# Defensive Testing Recommendations

Developers should include automated concurrency tests.

## Suggested Tests

| Test | Purpose |
|------|----------|
| Parallel reward requests | Verify single reward processing. |
| Duplicate POST requests | Verify idempotency. |
| High-frequency requests | Validate cooldown logic. |
| Concurrent authenticated sessions | Test shared state consistency. |

---

# Evidence Timeline

| Phase | Evidence |
|-------|----------|
| Enumeration | Dashboard review. |
| Workflow Analysis | Reward mechanism identified. |
| HTTP Interception | Reward endpoint captured. |
| Baseline Testing | Cooldown verified. |
| Repeater Testing | Request duplication performed. |
| Parallel Execution | Requests sent concurrently. |
| Validation | Whale Vault unlocked. |
| Completion | Flag evidence captured and redacted. |

---

# Lessons Learned

This room demonstrates several important web security concepts.

## Technical Lessons

- Business logic vulnerabilities are independent of input validation.
- Timing issues can bypass legitimate security controls.
- Race conditions often affect shared mutable state.
- Sequential replay testing is insufficient for concurrency vulnerabilities.
- Burp Repeater can simulate concurrent HTTP execution effectively.

---

## Defensive Lessons

- Critical state changes require atomic transactions.
- Idempotency prevents duplicate operations.
- Reward systems require concurrency testing.
- Logging should capture repeated authenticated state-changing requests.

---

# Portfolio Skills Demonstrated

Throughout this room I practiced:

- Web application enumeration.
- HTTP request interception.
- Burp Suite Repeater.
- Concurrent request testing.
- Race condition exploitation.
- Security documentation.
- Root cause analysis.
- Defensive remediation planning.

These skills are directly applicable to web application penetration testing and secure application design reviews.

---

# References & Further Reading

## Business Logic Security

- OWASP Business Logic Vulnerabilities
- OWASP Web Security Testing Guide
- PortSwigger Web Security Academy — Race Conditions
- Burp Suite Repeater Documentation
- CWE-362: Concurrent Execution using Shared Resource

These resources provide additional defensive guidance around concurrency vulnerabilities.

---

# Responsible Disclosure

This walkthrough documents activity performed **only within an authorized TryHackMe training environment**.

All testing was conducted against intentionally vulnerable infrastructure provided for cybersecurity education.

This repository is intended for:

- cybersecurity learning,
- technical documentation,
- penetration testing portfolio development,
- responsible security education.

No techniques demonstrated here should be used against systems without explicit authorization.

---

# Conclusion

**Towel on the Sunbed** highlights an important category of web application vulnerability that is frequently overlooked during traditional security testing.

Rather than exploiting malformed input, authentication bypasses, or server-side injections, the challenge focuses on **business logic integrity**. By replaying multiple authenticated reward requests concurrently, it was possible to bypass the intended cooldown mechanism and unlock privileged functionality significantly earlier than the application's designed workflow allowed.

This room reinforces the importance of testing **application state transitions under concurrent execution**. Security controls that work correctly during sequential interactions may still fail when multiple requests are processed simultaneously.

The assessment concluded with successful exploitation, verification of persistent application state changes, documentation of evidence, root cause analysis, and practical remediation recommendations while intentionally **redacting the final challenge flag** for responsible public sharing.

---

<div align="center">

### ⭐ Challenge Completed Successfully

**Business Logic • Race Condition • Burp Suite Repeater • Secure State Management**

*Prepared as a professional GitHub portfolio walkthrough.*

</div>
