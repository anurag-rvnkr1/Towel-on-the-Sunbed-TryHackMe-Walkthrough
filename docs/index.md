---
layout: default
title: "Towel on the Sunbed — TryHackMe Walkthrough"
description: "Professional GitHub Pages documentation for the Towel on the Sunbed TryHackMe challenge."
---

<div align="center">

# 🌞 Towel on the Sunbed

### Professional TryHackMe Walkthrough & Technical Documentation

**Hacker Holidays — Day 8**

*Business Logic Race Condition • Burp Suite Repeater • Web Application Security*

![TryHackMe](https://img.shields.io/badge/TryHackMe-Hacker%20Holidays-red?style=for-the-badge&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-success?style=for-the-badge)
![Category](https://img.shields.io/badge/Web-Business%20Logic-blue?style=for-the-badge)
![Vulnerability](https://img.shields.io/badge/Race%20Condition-Concurrency-orange?style=for-the-badge)
![Documentation](https://img.shields.io/badge/GitHub%20Pages-Portfolio-black?style=for-the-badge&logo=github)

---

*A portfolio-ready walkthrough documenting the complete exploitation of a business logic race condition inside a cryptocurrency staking application using Burp Suite Repeater.*

</div>

---

# 📌 About This Challenge

> **Towel on the Sunbed** is a web application challenge from the **TryHackMe Hacker Holidays** series that teaches how **race conditions** can compromise server-side business logic.

Unlike traditional web vulnerabilities, this challenge focuses on exploiting the application's workflow rather than manipulating user input. The target application awards cryptocurrency rewards every **24 hours**, and the objective is to unlock the **Whale Vault** by abusing concurrent reward processing.

This documentation demonstrates the complete methodology, technical analysis, screenshots, defensive recommendations, and security lessons learned during the assessment.

---

# 🎯 Challenge Overview

| Property | Details |
|----------|---------|
| **Platform** | TryHackMe |
| **Series** | Hacker Holidays |
| **Challenge** | Towel on the Sunbed |
| **Difficulty** | Easy |
| **Category** | Web Application Security |
| **Primary Vulnerability** | Business Logic Race Condition |
| **Tools Used** | Burp Suite Community Edition, Firefox |
| **Documentation Style** | Penetration Testing Portfolio |

---

# 🚀 Attack Chain

<div align="center">

```text
User Login
    │
    ▼
Portfolio Dashboard Enumeration
    │
    ▼
Capture Reward Request
    │
    ▼
Baseline Cooldown Validation
    │
    ▼
Duplicate Authenticated Requests
    │
    ▼
Group Requests in Burp Repeater
    │
    ▼
Send Requests in Parallel
    │
    ▼
Multiple Rewards Granted
    │
    ▼
Whale Vault Unlocked
    │
    ▼
Challenge Completed (Flag Redacted)
```

</div>

---

# 🛠️ Tools & Technologies

| Tool | Purpose |
|------|---------|
| 🛡️ **Burp Suite Proxy** | Intercept authenticated HTTP traffic. |
| 🔁 **Burp Repeater** | Replay and execute concurrent requests. |
| 🌐 **Firefox Browser** | Interact with the application through Burp. |
| 💻 **Linux Environment** | Testing workstation. |
| 🎯 **TryHackMe** | Authorized lab environment. |

---

# 🧠 Skills Demonstrated

<table>
<tr>
<td width="50%">

### Web Security

- Business Logic Testing
- HTTP Request Analysis
- Session-Aware Testing
- State Transition Validation
- Reward Workflow Enumeration

</td>
<td width="50%">

### Burp Suite

- Proxy Interception
- Request Replay
- Repeater Groups
- Parallel Request Execution
- Response Comparison

</td>
</tr>
</table>

---

# 📂 Documentation Navigation

| Section | Description |
|---------|-------------|
| **README.md** | Repository overview and walkthrough preview. |
| **Documentation/Documentation.md** | Complete technical penetration testing report. |
| **Documentation.docx** | Portfolio-ready report in Word format. |
| **Resources/notes.md** | Learning notes and methodology summary. |
| **docs/index.md** | GitHub Pages landing page (this page). |

---

# 🌐 Challenge Walkthrough

---

## Stage 1 — Initial Dashboard Enumeration

The application presents a cryptocurrency investment dashboard containing a staking system and a Whale Vault progression feature.

<p align="center">
<img src="assets/01-dashboard-initial.png" width="100%">
</p>

**Figure 1 — Initial cryptocurrency dashboard after authentication**

### Initial Observations

- Authenticated user dashboard.
- Cryptocurrency portfolio overview.
- Staking reward mechanism.
- Whale Vault locked until balance threshold is reached.

---

## Stage 2 — Understanding the Reward Workflow

The staking feature awards users **50 PONZI** tokens every **24 hours**.

<p align="center">
<img src="assets/02-dashboard-staking.png" width="100%">
</p>

**Figure 2 — Daily staking reward and Whale Vault requirement**

### Expected Workflow

| Day | Expected Balance |
|-----|------------------|
| Day 1 | 50 |
| Day 2 | 100 |
| Day 3 | 150 |
| Whale Vault | Unlocked |

The application's business logic assumes rewards cannot be claimed simultaneously.

---

## Stage 3 — Capturing the Reward Request

Burp Suite intercepts the authenticated reward request before it reaches the server.

<p align="center">
<img src="assets/03-burp-captured-claim.png" width="100%">
</p>

**Figure 3 — Captured HTTP reward request**

### Endpoint Analysis

```http
POST /claim
```

Key characteristics:

- Authenticated endpoint.
- Session cookie required.
- State-changing request.
- No payload manipulation needed.

---

## Stage 4 — Sending the Request to Burp Repeater

The intercepted request is transferred into Burp Repeater for controlled replay.

<p align="center">
<img src="assets/04-repeater-request.png" width="100%">
</p>

**Figure 4 — Reward request inside Burp Repeater**

### Purpose

- Preserve authentication.
- Duplicate identical requests.
- Test concurrent execution.

---

## Stage 5 — Creating Parallel Request Groups

Several identical authenticated requests are grouped together.

<p align="center">
<img src="assets/05-repeater-tab-group.png" width="100%">
</p>

**Figure 5 — Request grouping for race condition testing**

The request group ensures multiple requests are transmitted at nearly the same moment.

---

## Stage 6 — Triggering the Race Condition

Burp Suite's **Send Group in Parallel** feature dispatches all grouped requests simultaneously.

<p align="center">
<img src="assets/06-repeater-parallel-send.png" width="100%">
</p>

**Figure 6 — Concurrent execution using Burp Repeater**

### Testing Objective

Determine whether multiple authenticated requests can pass reward validation before cooldown state is updated.

---

## Stage 7 — Observing Concurrent Responses

Multiple reward requests receive successful responses during the same execution window.

<p align="center">
<img src="assets/07-parallel-responses.png" width="100%">
</p>

**Figure 7 — Multiple successful reward responses**

### Security Observation

Instead of rejecting duplicate reward claims, the application credits several rewards concurrently.

This behavior confirms a **business logic race condition**.

---

## Stage 8 — Whale Vault Unlock

Refreshing the application validates that the balance has been permanently updated.

<p align="center">
<img src="assets/08-whale-vault-unlocked.png" width="100%">
</p>

**Figure 8 — Whale Vault unlocked after successful race condition**

The reward threshold is reached without waiting multiple days.

---

## Stage 9 — Completion Evidence

The Whale Vault reveals the challenge completion page.

<p align="center">
<img src="assets/09-flag-redacted.png" width="100%">
</p>

**Figure 9 — Final flag evidence (redacted)**

> 🔒 The challenge flag is intentionally hidden to preserve the learning experience.

---

## Stage 10 — Final Application State

<p align="center">
<img src="assets/10-whale-vault-final.png" width="100%">
</p>

**Figure 10 — Completed Whale Vault state**

---

# 🔍 Technical Deep Dive

## Race Condition Explained

### Expected Secure Flow

```text
Check cooldown
      │
      ▼
Grant reward
      │
      ▼
Update cooldown
```

Only one request succeeds.

---

### Vulnerable Flow

```text
Request A
Request B
Request C
      │
      ▼
Eligibility checked simultaneously
      ▼
Multiple rewards granted
      ▼
Cooldown updated afterwards
```

The server validates multiple requests before committing shared account state.

---

# ⚙️ Root Cause Analysis

The application's reward endpoint performs **validation** and **state updates** separately.

### Missing Atomic Transaction

```python
if reward_available(user):
    add_reward(user)
    update_cooldown(user)
```

Multiple requests evaluate `reward_available()` before `update_cooldown()` completes.

---

## Why Sequential Replay Failed

Sequential replay updates cooldown before the next request arrives.

Parallel replay allows several requests to enter the same validation window.

---

# 💥 Security Impact

| Impact Area | Description |
|-------------|-------------|
| **Integrity** | Duplicate reward credits modify account balance. |
| **Business Logic** | Intended cooldown enforcement bypassed. |
| **Privilege Progression** | Whale Vault unlocked earlier than intended. |
| **Financial Risk** | Similar flaws could affect cashback, loyalty, or cryptocurrency systems. |

---

# 🛡️ Defensive Recommendations

## Atomic Transactions

Ensure validation and reward updates occur within a single transaction.

## Row-Level Locking

Prevent concurrent updates to the same account record.

## Idempotency Keys

Reject duplicate reward requests processed within the same reward period.

## Concurrency Testing

Include automated parallel request testing during QA and security testing.

## Audit Logging

Log repeated state-changing requests for anomaly detection.

---

# 🔵 Blue Team Detection Opportunities

Potential detection signals include:

| Indicator | Detection Opportunity |
|-----------|----------------------|
| Same session sending `/claim` repeatedly | High-confidence alert. |
| Multiple successful reward events within milliseconds | Business logic abuse detection. |
| Rapid balance growth | Financial integrity monitoring. |
| Concurrent authenticated POST requests | Suspicious session behavior. |

---

# 🧬 MITRE ATT&CK Mapping *(Educational)*

| Technique | Relevance |
|-----------|-----------|
| **Valid Accounts (T1078)** | Uses legitimate authenticated session. |
| **Application Layer Protocol (T1071)** | HTTP used for exploitation. |
| **Exploitation of Public-Facing Application** | Vulnerable reward endpoint abused. |

---

# 📚 Learning Outcomes

After completing this room, I strengthened practical knowledge in:

- Business Logic Vulnerabilities
- Race Condition Identification
- Burp Suite Repeater
- Concurrent HTTP Request Testing
- State Transition Analysis
- Secure Reward Workflow Design
- Technical Penetration Testing Documentation

---

# 📊 Evidence Summary

| Screenshot | Description |
|------------|-------------|
| `01-dashboard-initial.png` | Dashboard after login. |
| `02-dashboard-staking.png` | Reward workflow. |
| `03-burp-captured-claim.png` | Captured reward request. |
| `04-repeater-request.png` | Request moved into Repeater. |
| `05-repeater-tab-group.png` | Request grouping. |
| `06-repeater-parallel-send.png` | Parallel request execution. |
| `07-parallel-responses.png` | Successful concurrent responses. |
| `08-whale-vault-unlocked.png` | Whale Vault unlocked. |
| `09-flag-redacted.png` | Completion evidence. |
| `10-whale-vault-final.png` | Final application state. |

---

# 📁 Repository Structure

```text
Towel-on-the-Sunbed-TryHackMe-Walkthrough/
├── Documentation/
│   ├── Documentation.md
│   └── Documentation.docx
├── Resources/
│   └── notes.md
├── Screenshots/
├── docs/
│   ├── index.md
│   └── assets/
└── README.md
```

---

# ⚠️ Responsible Disclosure

This walkthrough documents activities performed **only within the authorized TryHackMe training environment**.

The repository is intended for:

- Cybersecurity education.
- Penetration testing practice.
- Documentation portfolio development.
- Secure software development awareness.

No techniques demonstrated here should be used against systems without explicit authorization.

---

<div align="center">

# ⭐ Thank You for Visiting

### Towel on the Sunbed — TryHackMe Walkthrough

**Business Logic • Race Condition • Burp Suite Repeater**

Cybersecurity Portfolio by **Anurag Revankar**

*Building practical cybersecurity knowledge through hands-on labs, professional documentation, and responsible security research.*

</div>
