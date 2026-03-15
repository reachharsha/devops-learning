---
render_with_liquid: false
---
# 06 — Metrics & Issues: Reading Sonar Like a Pro

A Sonar dashboard can look intimidating at first. This lesson teaches you what matters and what to ignore.

---

## 🎯 Learning Objectives

- Interpret the main metrics (reliability, security, maintainability)
- Understand issue categories and severities
- Know how to prioritize fixes
- Avoid “metric theater” (numbers without outcomes)

---

## 1) Issue Types (The Big Four)

### Bugs (Reliability)
Likely to cause incorrect behavior (or crash).

### Vulnerabilities (Security)
Patterns that can be exploited.

### Security Hotspots
Potentially risky patterns that require **human review**.

### Code Smells (Maintainability)
Code that is harder to change safely (complexity, duplication, unclear logic).

---

## 2) Severity vs Priority

Sonar reports severity (Blocker → Info), but your team should define priority.

A practical priority model:
- **P0**: exploitable vulnerability, auth bypass, secrets exposure
- **P1**: critical bug in a hot path, major reliability issues
- **P2**: maintainability issues that will slow the next refactor
- **P3**: minor stylistic smells (fix opportunistically)

---

## 3) Ratings and Technical Debt (Use Carefully)

Sonar often summarizes maintainability with:
- Ratings (A–E)
- Estimated “debt” time

These are helpful trends, but don’t treat them as exact accounting.

Use them for:
- comparing modules
- tracking trend over time
- deciding where to invest cleanup

---

## 4) Duplications

Duplicated code increases:
- bug surface area
- cost of changes

But duplication isn’t always bad (some duplication is intentional to decouple).

Rule of thumb:
- Reduce duplication in shared logic
- Avoid premature “DRY” that harms readability

---

## 5) The Most Important Filter: New Code

If your repo has many issues, focus on:
- **New Code** issues
- Highest severity security/reliability

This keeps Sonar from becoming unmaintainable.

---

## ✅ Hands-on Lab: Triage Like a Team

1. Open your project dashboard.
2. Filter to **New Code**.
3. Pick 5 issues:
   - 2 security
   - 2 bugs
   - 1 maintainability
4. For each issue, write:
   - Why it matters
   - Whether it’s real or false positive
   - The smallest safe fix

---

## Quick Check

1. What’s the difference between a vulnerability and a security hotspot?
2. Why is “new code” the key adoption tool?
3. When can duplication be acceptable?

---

## Next

Go to **Lesson 07**: Quality Gates.
