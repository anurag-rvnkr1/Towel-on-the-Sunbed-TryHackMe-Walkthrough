# Towel on the Sunbed — TryHackMe Walkthrough

## 1. Executive Summary

**Platform:** TryHackMe  
**Challenge:** Towel on the Sunbed  
**Primary category:** Web / Business Logic  
**Key vulnerability:** Race condition  
**Primary tool:** Burp Suite Repeater

This walkthrough documents the full assessment flow from application review through controlled concurrent request testing and final vault verification.

> **Public flag handling:** the final flag is intentionally redacted.

---

## 2. Scenario

The target application presents a cryptocurrency portfolio dashboard with a staking feature.

The visible workflow indicates that:

- each successful staking claim provides **50 PONZI**;
- a cooldown is enforced between claims;
- the Whale Vault requires **150 PONZI**.

The intended workflow therefore requires waiting between successful reward claims.

---

## 3. Initial Application Review

After accessing the dashboard, the initial balance and available staking action were recorded.

![Initial dashboard](../docs/assets/01-dashboard-initial.png)

**Figure 1 — Initial application dashboard**

The dashboard exposes market prices, a staking-reward panel, and the Whale Vault progression.

![Staking state](../docs/assets/02-dashboard-staking.png)

**Figure 2 — Staking reward panel and Whale Vault requirement**

The first step in business-logic testing is establishing the legitimate baseline before attempting to manipulate timing.

---

## 4. Capturing the Reward Endpoint

Burp Suite was placed between the browser and the lab application. Interception was enabled and the **Claim Reward** action was triggered.

![Captured request](../docs/assets/03-burp-captured-claim.png)

**Figure 3 — Intercepted staking claim request**

The request uses the authenticated browser session and a simple state-changing endpoint:

```http
POST /claim HTTP/1.1
Host: <LAB_HOST>
Cookie: connect.sid=<SESSION>
Content-Length: 0
```

The actual host and session value are omitted from this public report.

---

## 5. Baseline Behavior

A single claim was sent normally.

The response confirmed a successful staking operation and showed the reward amount. An immediate sequential replay was then tested.

The expected behavior was observed:

```text
First claim  → accepted
Immediate replay → cooldown / rate limit
```

This baseline established that the endpoint was intended to prevent rapid repeated claims.

---

## 6. Preparing the Race Test

The captured request was transferred to Burp Suite Repeater.

![Repeater request](../docs/assets/04-repeater-request.png)

**Figure 4 — Claim request prepared for controlled replay**

The request was duplicated to create several equivalent requests using the same authenticated context.

---

## 7. Grouping Requests

The duplicated Repeater tabs were placed into a single request group.

![Repeater group](../docs/assets/05-repeater-tab-group.png)

**Figure 5 — Grouped Repeater requests**

Grouping makes it possible to launch the requests with coordinated timing, which is essential when testing for a race condition.

---

## 8. Parallel Execution

Burp Suite Repeater provides a parallel group-send option. The requests were dispatched concurrently.

![Parallel send](../docs/assets/06-repeater-parallel-send.png)

**Figure 6 — Concurrent execution of grouped requests**

The test was designed to answer a specific question:

> Can multiple requests pass the cooldown/eligibility check before the account state is updated consistently?

This is preferable to blind repetition because it isolates the concurrency condition that the vulnerability depends on.

---

## 9. Race Condition Observed

The resulting responses demonstrated that more than one claim could be accepted during the same concurrent execution window.

![Parallel responses](../docs/assets/07-parallel-responses.png)

**Figure 7 — Multiple accepted reward responses**

The observed sequence is conceptually:

```text
Request A ─┐
Request B ─┼──> concurrent eligibility check ──> multiple accepted updates
Request C ─┘
```

Under normal serialized execution, only the first request should modify the balance while later requests observe the cooldown.

The concurrent test instead allowed multiple state transitions.

---

## 10. Application-Level Verification

With the interception state disabled, the dashboard was refreshed to confirm the resulting account state.

![Whale Vault unlocked](../docs/assets/08-whale-vault-unlocked.png)

**Figure 8 — Whale Vault unlocked after the race**

The balance had reached the threshold required to activate the vault without following the intended waiting period.

This second-stage verification is important: it demonstrates that the race condition affected persistent application state rather than only producing unusual HTTP responses.

---

## 11. Final Evidence and Flag Handling

Opening the unlocked vault reveals the challenge's final flag.

For this public portfolio repository, the flag value is intentionally hidden.

![Redacted flag evidence](../docs/assets/09-flag-redacted.png)

**Figure 9 — Final flag evidence with the challenge value redacted**

The final state was captured separately to document successful completion without publishing the answer.

![Final state](../docs/assets/10-whale-vault-final.png)

**Figure 10 — Final completed application state**

---

## 12. Technical Root Cause

The underlying issue is a failure to make the claim operation atomic.

A secure implementation should conceptually perform:

```text
1. Check eligibility
2. Check cooldown
3. Update balance
4. Record successful claim time
```

as one protected state transition.

The vulnerable pattern permits overlapping requests to perform the checks before the shared account state is fully committed.

### Expected

```text
Request 1 → check → update → success
Request 2 → check current state → cooldown → reject
```

### Vulnerable

```text
Request 1 ─┐
Request 2 ─┼→ check stale/unchanged state
Request 3 ─┘
           ↓
      multiple updates
```

This is a classic concurrency problem in which correctness depends on operations being serialized or transactionally isolated.

---

## 13. Security Impact

A similar flaw in a production rewards or financial system could lead to:

- duplicate credits;
- unauthorized balance inflation;
- bypass of transaction or reward limits;
- incorrect account state;
- financial loss;
- downstream abuse if the inflated balance can be redeemed or withdrawn.

The actual impact depends on what the reward represents and on controls implemented elsewhere in the system.

---

## 14. Recommended Remediation

### Atomic state transitions

Perform eligibility checks and balance changes within one atomic transaction.

### Concurrency control

Use appropriate transaction isolation, row-level locking, or equivalent synchronization for shared account state.

### Idempotency

Give each logical reward operation a unique identifier and reject duplicate processing.

### Server-side enforcement

Keep cooldown timestamps and reward eligibility decisions on the server.

### Database invariants

Use constraints and transactional updates to protect critical balance rules.

### Concurrency regression tests

Automated tests should issue simultaneous claims and verify that only one state transition succeeds.

---

## 15. Security Testing Lessons

This challenge demonstrates why web testing should go beyond checking for common payload-based vulnerabilities.

A complete assessment of a state-changing endpoint should consider:

```text
Authentication
Authorization
Input validation
State transitions
Timing
Concurrency
Replay behavior
Idempotency
```

A feature can appear correctly protected during normal sequential use and still fail under concurrent access.

---

## 16. Evidence Index

| Figure | Evidence | Filename |
|---|---|---|
| 1 | Initial dashboard | `01-dashboard-initial.png` |
| 2 | Staking workflow | `02-dashboard-staking.png` |
| 3 | Captured request | `03-burp-captured-claim.png` |
| 4 | Repeater request | `04-repeater-request.png` |
| 5 | Grouped requests | `05-repeater-tab-group.png` |
| 6 | Parallel execution | `06-repeater-parallel-send.png` |
| 7 | Parallel responses | `07-parallel-responses.png` |
| 8 | Vault unlocked | `08-whale-vault-unlocked.png` |
| 9 | Redacted flag evidence | `09-flag-redacted.png` |
| 10 | Final state | `10-whale-vault-final.png` |

---

## 17. Conclusion

The Towel on the Sunbed challenge demonstrates that business logic can fail even when a visible cooldown appears to work correctly.

The key workflow was:

```text
Dashboard review
      ↓
Capture POST /claim
      ↓
Establish sequential baseline
      ↓
Duplicate request
      ↓
Group requests
      ↓
Send concurrently
      ↓
Observe multiple accepted claims
      ↓
Verify balance change
      ↓
Unlock Whale Vault
```

The central lesson is that security controls protecting shared state must remain correct under concurrent execution, not merely under ordinary sequential use.

---

## 18. Responsible Use

This walkthrough was created for an authorized TryHackMe training environment and cybersecurity education. Techniques should only be reproduced against assets for which explicit authorization has been granted.

**Author:** Anurag
