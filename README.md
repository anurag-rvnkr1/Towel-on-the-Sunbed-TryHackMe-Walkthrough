# 🏖️ Towel on the Sunbed — TryHackMe Walkthrough

<div align="center">

# 🌞 Towel on the Sunbed

### *TryHackMe Hacker Holidays — Day 8 Walkthrough*

<img src="docs/assets/00-room-banner.png" width="100%" alt="Towel on the Sunbed Banner"/>

![TryHackMe](https://img.shields.io/badge/TryHackMe-Hacker%20Holidays-red?style=for-the-badge\&logo=tryhackme)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy-success?style=for-the-badge)
![Category](https://img.shields.io/badge/Category-Business%20Logic%20%7C%20Race%20Condition-blue?style=for-the-badge)
![Burp Suite](https://img.shields.io/badge/Burp%20Suite-Repeater-orange?style=for-the-badge\&logo=burpsuite)
![Status](https://img.shields.io/badge/Writeup-Complete-darkgreen?style=for-the-badge)

*A professional cybersecurity walkthrough documenting the discovery and exploitation of a business logic race condition in a cryptocurrency staking application using Burp Suite Repeater.*

</div>

---

## 📖 Overview

**Towel on the Sunbed** is one of the web application challenges from **TryHackMe's Hacker Holidays** event. Rather than exploiting injection flaws or authentication bypasses, this room demonstrates a **business logic vulnerability** caused by improper handling of concurrent requests.

The application rewards users with cryptocurrency every **24 hours** through a staking mechanism. Under normal circumstances, users must wait several days before unlocking the **Whale Vault**. By analyzing the application's workflow and sending multiple authenticated requests simultaneously, the intended cooldown mechanism can be bypassed.

This repository documents the complete assessment methodology, evidence, technical analysis, and defensive recommendations.

> **Focus Area:** Business Logic Vulnerabilities • Race Conditions • Burp Suite Repeater • Secure State Management

---

# 🎯 Objectives

During this lab, the following objectives were completed:

* Analyze the application's reward mechanism.
* Capture authenticated HTTP requests.
* Understand expected cooldown behavior.
* Identify a race condition opportunity.
* Execute parallel HTTP requests using Burp Suite Repeater.
* Trigger multiple reward claims simultaneously.
* Unlock the Whale Vault through business logic abuse.
* Document evidence and mitigation strategies professionally.

---

# 🛠️ Skills Demonstrated

<table>
<tr><td>🌐 Web Application Assessment</td><td>Analyzing client/server business workflows.</td></tr>
<tr><td>🕵️ Burp Suite Proxy</td><td>Intercepting authenticated HTTP traffic.</td></tr>
<tr><td>🔁 Burp Repeater</td><td>Duplicating and replaying HTTP requests.</td></tr>
<tr><td>⚡ Race Condition Testing</td><td>Sending grouped concurrent requests.</td></tr>
<tr><td>🔍 Business Logic Analysis</td><td>Understanding state transition vulnerabilities.</td></tr>
<tr><td>📑 Technical Documentation</td><td>Professional penetration testing reporting.</td></tr>
</table>

---

# 🧰 Tools Used

| Tool                             | Purpose                                     |
| -------------------------------- | ------------------------------------------- |
| **TryHackMe**                    | Authorized cybersecurity training platform. |
| **Burp Suite Community Edition** | HTTP interception and request manipulation. |
| **Burp Repeater**                | Concurrent request testing.                 |
| **Firefox**                      | Browser configured through Burp proxy.      |
| **Linux**                        | Testing environment.                        |

---

# 📂 Repository Structure

```text
Towel-on-the-Sunbed-TryHackMe-Walkthrough/
│
├── README.md                         # Main project overview
├── Documentation/
│   ├── Documentation.md              # Detailed technical walkthrough
│   └── Documentation.docx            # Portfolio report (Word)
│
├── Resources/
│   └── notes.md                      # Quick methodology & learning notes
│
├── Screenshots/
│   ├── 01-dashboard-initial.png
│   ├── 02-dashboard-staking.png
│   ├── 03-burp-captured-claim.png
│   ├── 04-repeater-request.png
│   ├── 05-repeater-tab-group.png
│   ├── 06-repeater-parallel-send.png
│   ├── 07-parallel-responses.png
│   ├── 08-whale-vault-unlocked.png
│   ├── 09-flag-redacted.png
│   └── 10-whale-vault-final.png
│
├── docs/
│   ├── index.md                      # GitHub Pages documentation
│   └── assets/                       # Images used by GitHub Pages
│
├── .github/
├── _config.yml
└── .gitignore
```

---

# 🚀 Attack Chain Overview

<div align="center">

```text
User Registration
        │
        ▼
Access Portfolio Dashboard
        │
        ▼
Capture POST /claim Request
        │
        ▼
Verify Cooldown Behaviour
        │
        ▼
Duplicate Request in Repeater
        │
        ▼
Create Parallel Request Group
        │
        ▼
Send Concurrent Requests
        │
        ▼
Multiple Rewards Credited
        │
        ▼
Whale Vault Unlocks
        │
        ▼
Capture Completion Evidence
```

</div>

---

# 🔍 Vulnerability Summary

| Item                       | Details                                               |
| -------------------------- | ----------------------------------------------------- |
| **Challenge Category**     | Business Logic                                        |
| **Vulnerability**          | Race Condition                                        |
| **Affected Functionality** | Cryptocurrency staking reward endpoint                |
| **Impact**                 | Multiple reward claims during cooldown period         |
| **Root Cause**             | Non-atomic state transition under concurrent requests |

The vulnerability is not caused by malformed input. Instead, it occurs because several requests are processed **before the application updates shared reward state**.

---

# 📸 Walkthrough Preview

## Stage 1 — Initial Dashboard

The application provides a cryptocurrency dashboard showing portfolio balance, staking rewards, and Whale Vault progression.

<p align="center">
<img src="docs/assets/01-dashboard-initial.png" width="90%">
</p>

---

## Stage 2 — Understanding the Reward Mechanism

The staking feature grants cryptocurrency rewards after a cooldown period.

<p align="center">
<img src="docs/assets/02-dashboard-staking.png" width="90%">
</p>

---

## Stage 3 — Capturing the HTTP Request

Burp Suite intercepts the authenticated reward request before it reaches the server.

<p align="center">
<img src="docs/assets/03-burp-captured-claim.png" width="90%">
</p>

---

## Stage 4 — Sending the Request to Repeater

The captured request is moved into Burp Repeater for controlled replay.

<p align="center">
<img src="docs/assets/04-repeater-request.png" width="90%">
</p>

---

## Stage 5 — Building a Parallel Request Group

Multiple identical requests are grouped together.

<p align="center">
<img src="docs/assets/05-repeater-tab-group.png" width="90%">
</p>

---

## Stage 6 — Triggering the Race Condition

Burp Suite dispatches every grouped request simultaneously.

<p align="center">
<img src="docs/assets/06-repeater-parallel-send.png" width="90%">
</p>

---

## Stage 7 — Observing Multiple Successful Rewards

Several concurrent requests receive successful responses before cooldown enforcement occurs.

<p align="center">
<img src="docs/assets/07-parallel-responses.png" width="90%">
</p>

---

## Stage 8 — Whale Vault Unlock

Refreshing the dashboard confirms the account balance exceeded the intended threshold.

<p align="center">
<img src="docs/assets/08-whale-vault-unlocked.png" width="90%">
</p>

---

## Stage 9 — Completion Evidence

The challenge completes after opening the Whale Vault.

<p align="center">
<img src="docs/assets/10-whale-vault-final.png" width="90%">
</p>

> 🔒 **The final flag has been intentionally redacted in this repository.**

---

# 🧠 Technical Analysis

## Why the Race Condition Exists

The reward endpoint performs eligibility checks before recording the reward claim.

### Expected Behaviour

```text
Check cooldown
      │
      ▼
Reward granted
      │
      ▼
Cooldown updated
```

### Vulnerable Behaviour

```text
Request A ─┐
Request B ─┼── Eligibility checked simultaneously
Request C ─┘
      │
      ▼
Multiple reward updates committed
```

Because the state update is not atomic, multiple requests observe the same eligible state.

---

# 🛡️ Security Impact

If implemented in a production financial platform, this flaw could result in:

* Duplicate cryptocurrency rewards.
* Inflation of user balances.
* Reward farming.
* Financial abuse.
* Business logic bypass.
* Incorrect transaction accounting.

Race conditions frequently affect systems handling:

* Wallet balances.
* Loyalty rewards.
* Coupons.
* Gift cards.
* Banking transactions.
* Inventory reservations.

---

# ✅ Recommended Mitigations

| Recommendation                  | Purpose                                  |
| ------------------------------- | ---------------------------------------- |
| Atomic database transactions    | Prevent concurrent balance modification. |
| Row-level locking               | Serialize updates for a single account.  |
| Idempotency tokens              | Reject duplicate reward operations.      |
| Server-side cooldown validation | Prevent replay attacks.                  |
| Transaction isolation           | Ensure consistent reads/writes.          |
| Concurrency testing             | Detect race conditions during QA.        |

---

# 📚 Key Learning Outcomes

After completing this room, I learned how to:

* Identify business logic vulnerabilities.
* Test server-side state transitions.
* Capture authenticated requests safely.
* Use Burp Suite Repeater effectively.
* Perform concurrent HTTP testing.
* Validate application state after exploitation.
* Produce professional penetration testing documentation.

---

# 📖 Documentation Included

| File                   | Description                               |
| ---------------------- | ----------------------------------------- |
| **README.md**          | Project overview and walkthrough preview. |
| **Documentation.md**   | Complete technical write-up.              |
| **Documentation.docx** | Portfolio-ready report.                   |
| **Resources/notes.md** | Quick notes and methodology.              |
| **docs/index.md**      | GitHub Pages documentation website.       |

---

# 🌐 GitHub Pages

This repository includes a dedicated GitHub Pages site for portfolio presentation.

**Features include:**

* Professional landing page.
* Attack chain visualization.
* Screenshot gallery.
* Vulnerability explanation.
* Mitigation section.
* Learning outcomes.
* Mobile-friendly documentation.

---

# ⚠️ Responsible Disclosure

This repository documents techniques performed **only inside an authorized TryHackMe lab** for educational purposes.

No techniques demonstrated here should be used against systems without explicit permission.

---

# 👨‍💻 Author

**Anurag Revankar**

Cybersecurity Enthusiast • Web Application Security • Penetration Testing • SOC & Blue Team Learning

GitHub: **anurag-rvnkr1**

---

<div align="center">

### ⭐ If you found this walkthrough useful, consider starring the repository.

**Built for learning, documentation, and cybersecurity portfolio development.**

</div>
