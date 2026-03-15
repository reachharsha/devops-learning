---
render_with_liquid: false
---
# 07 — Quality Gates (Make Quality Enforceable)

A **Quality Gate** turns Sonar from a dashboard into a **policy**.

When used correctly, a gate:
- blocks risky changes
- encourages good habits
- keeps teams from drowning in legacy debt

---

## 🎯 Learning Objectives

- Understand what a Quality Gate is
- Create a practical “new code” gate
- Fail CI when the gate fails
- Avoid gate designs that teams will bypass

---

## 1) What a Quality Gate Is

A Quality Gate is a set of conditions like:
- New bugs = 0
- New vulnerabilities = 0
- Coverage on new code ≥ 80%

If any condition fails, the gate fails.

---

## 2) The Most Practical Gate: New Code Gate

For most teams (especially with legacy code), this is the best starting gate:

Suggested gate (starter):
- New vulnerabilities: **0**
- New bugs: **0**
- New code smells: keep low (or a small threshold)
- Coverage on new code: **≥ 80%** (adjust for your reality)

Why it works:
- prevents backsliding
- doesn’t require cleaning the entire legacy base first

---

## 3) “New Code” Definition

Quality Gate results depend on what “new code” means.

Common definitions:
- Since a specific date
- Since previous version/release
- Since the branch was created (PR analysis)

Pick one and make it consistent.

---

## 4) Gate Anti-Patterns

### Anti-pattern A: Gate is too strict for reality
Result: teams disable Sonar.

### Anti-pattern B: Gate checks only maintainability
Result: security/reliability regressions slip.

### Anti-pattern C: Gate checks everything, not just new code
Result: legacy blocks delivery.

---

## ✅ Hands-on Lab: Build a Gate You Can Enforce

In Sonar UI:
1. Go to **Quality Gates**
2. Copy the default gate to “Team New Code Gate”
3. Add/adjust:
   - New Bugs = 0
   - New Vulnerabilities = 0
   - Coverage on New Code ≥ 80% (or a target you can meet)
4. Assign the gate to your project
5. Run analysis and check gate status

---

## Quick Check

1. Why is a “new code” gate often better than an “entire codebase” gate?
2. What happens when a gate fails in a healthy DevOps workflow?
3. Name one gate anti-pattern.

---

## Next

Go to **Lesson 08**: [Coverage & test reports](08-tests-and-coverage.md).
