---
render_with_liquid: false
---
# 16 — Triage Workflow (How Teams Actually Use Sonar)

Sonar succeeds when it becomes part of daily engineering—not a quarterly cleanup.

This lesson gives you a team workflow that scales.

---

## 🎯 Learning Objectives

- Triage issues efficiently
- Focus on new code while paying down legacy debt
- Build a simple ownership model
- Connect Sonar issues to delivery (PRs, tickets)

---

## 1) The “New Code First” Workflow

Daily:
- PR introduces new issue → fix before merge

Weekly:
- Allocate time to pay down top legacy issues in risky hotspots

Monthly:
- Review trends and adjust gates/profiles if necessary

---

## 2) Issue Lifecycle (Practical)

Most teams need these states:
- **Open**: needs attention
- **Confirmed**: real issue
- **False Positive**: rule triggered incorrectly
- **Won’t Fix**: understood risk accepted (rare; document why)

Use “Won’t Fix” sparingly; otherwise it becomes a trash bin.

---

## 3) Ownership

Good ownership patterns:
- Per-service ownership (microservices)
- CODEOWNERS + PR checks
- A security champion rotation for hotspots

Avoid “everyone owns everything”. It turns into “no one owns anything”.

---

## 4) Prioritization Heuristics

Fix first:
- New vulnerabilities
- New bugs in hot paths
- Issues in auth/payment/data access code

Fix next:
- high-severity maintainability problems that block refactors

Fix later:
- low-value stylistic smells

---

## ✅ Hands-on Lab

1. Create a simple policy for your repo:
   - New vulnerabilities = 0
   - New bugs = 0
2. Pick a legacy issue and decide:
   - Fix now
   - Create a ticket
   - Mark false positive
   - Mark won’t fix (only with justification)
3. Write 3 triage rules you will follow in PR reviews.

---

## Quick Check

1. Why is ownership important for Sonar adoption?
2. When is “Won’t Fix” acceptable?
3. What kinds of issues should be P0?

---

## Next

Go to **Lesson 17**: false positives and tuning.
