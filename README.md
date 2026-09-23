# 🏖️ Towel on the Sunbed — TryHackMe Walkthrough

> **Day 8 | Hacker Holidays | Business Logic / Race Condition**

Portfolio documentation for the TryHackMe **Towel on the Sunbed** challenge. The lab demonstrates how a cryptocurrency staking endpoint can be abused when concurrent reward claims are processed without an atomic state transition.

## 🎯 Learning Objectives

- Understand business-logic vulnerabilities
- Establish normal application behavior before testing
- Capture authenticated HTTP requests with Burp Suite
- Reproduce a race condition with Repeater
- Use grouped parallel requests for controlled concurrency testing
- Validate state changes through the application UI
- Document findings and remediation professionally

## 🧰 Tooling

| Tool | Role |
|---|---|
| TryHackMe | Authorized training environment |
| Firefox | Web application interaction |
| Burp Suite | Proxy/interception |
| Repeater | Request duplication and parallel testing |

## 🔬 Core Finding

The staking endpoint enforces a cooldown between reward claims. A normal claim succeeds, while an immediate sequential retry is rejected.

The interesting behavior appears when equivalent requests are dispatched concurrently. Multiple requests can pass the eligibility check during the same race window, causing the account balance to increase more than once.

This is a **race condition in business logic** rather than a conventional injection flaw.

## 🗂️ Repository Structure

```text
Towel-on-the-Sunbed-TryHackMe-Walkthrough/
├── Documentation/
│   └── Documentation.md
├── Resources/
│   └── notes.md
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
├── docs/
│   ├── index.md
│   └── assets/
└── README.md
```

## 📚 Full Walkthrough

Read the complete technical report in [`Documentation/Documentation.md`](Documentation/Documentation.md).

A GitHub Pages version is available at [`docs/index.md`](docs/index.md).

## 🔐 Flag Policy

The final challenge flag is intentionally **redacted** in the public documentation. The purpose is to preserve the evidence and methodology without publishing a direct copyable answer.

## ⚠️ Scope

All testing described here is for the authorized TryHackMe lab. Do not reproduce these actions against systems without explicit authorization.

---
**Author:** Anurag  
**Challenge:** Towel on the Sunbed  
**Focus:** Web Security • Business Logic • Race Conditions
