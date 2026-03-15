---
render_with_liquid: false
---
# 17 — False Positives, Suppression, and Rule Tuning

No static analysis tool is perfect.

Your job is to build a workflow where:
- true positives are fixed
- false positives don’t create noise
- the rule set stays stable

---

## 🎯 Learning Objectives

- Handle false positives without disabling the entire tool
- Learn safe suppression strategies
- Tune profiles instead of fighting the same issues repeatedly

---

## 1) Identify a False Positive

A false positive typically looks like:
- the rule triggers, but the code is safe in your context
- the rule lacks enough context to infer control flow

Before marking false positive:
- confirm you understand the rule’s intent
- check whether a small refactor would make the intent obvious

---

## 2) Suppression Strategy (Least Dangerous First)

Recommended order:
1. Refactor code to make it clearer (often fixes the issue for real)
2. Tune rule parameters in the Quality Profile
3. Mark a specific issue as false positive
4. Use exclusions only when necessary (generated code, third-party)

Avoid broad exclusions like excluding all `src/` or entire modules.

---

## 3) Generated Code and Vendor Code

Do not waste time triaging findings in:
- generated clients
- vendored libraries
- build outputs

Exclude them at the project config level.

---

## 4) Keep a “Rule Change Log” (Team Habit)

When you change rules/profiles:
- document why
- announce it
- avoid frequent churn

This prevents “why did our pipeline start failing today?” incidents.

---

## ✅ Hands-on Lab

1. Find 1 issue you suspect is false positive.
2. Try to refactor to satisfy the rule.
3. If it’s still false positive:
   - mark it in Sonar
   - document why in your PR description
4. Identify one noisy rule and tune its parameter in your profile.

---

## Quick Check

1. What’s the safest way to handle noisy rules?
2. Why are broad exclusions risky?
3. Why should rule changes be documented?

---

## Next

Go to **Lesson 18**: governance and organization-wide standards.
