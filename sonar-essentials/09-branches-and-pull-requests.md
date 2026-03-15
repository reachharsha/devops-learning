---
render_with_liquid: false
---
# 09 — Branches & Pull Requests (Shift-Left Quality)

The highest ROI Sonar workflow is:
- analyze every PR
- comment on problems before merge
- enforce a Quality Gate on **new code**

---

## 🎯 Learning Objectives

- Understand branch vs PR analysis
- Learn PR decoration (comments/status checks)
- Know the typical CI inputs for PR analysis
- Understand feature availability differences (SonarQube editions vs SonarCloud)

---

## 1) Branch Analysis vs PR Analysis

### Branch analysis
- Tracks long-lived branches (e.g., `main`, `develop`, release branches)
- Shows trends over time

### PR analysis
- Focuses on the delta introduced by a PR
- Ideal for catching issues before merge

---

## 2) PR Decoration

PR decoration means Sonar results show up in your Git provider:
- status checks (pass/fail)
- inline comments (depending on integration)
- summary metrics on the PR

This keeps feedback where developers work.

---

## 3) Typical Inputs for PR Analysis

Most CI systems pass:
- PR key/number
- source branch
- target branch
- repo identifier

The exact method depends on platform and scanner.

---

## 4) Feature Availability Note

Depending on your setup:
- **SonarCloud** commonly supports PR analysis workflows with Git providers.
- **SonarQube** feature availability can vary by edition and configuration.

If a feature is missing in your UI (branches/PRs), check the edition/docs and your integration settings.

---

## ✅ Hands-on Lab (Conceptual)

Even if you don’t configure full PR decoration yet:
1. Create a branch with a small code change that introduces a code smell.
2. Run analysis for that branch.
3. Observe whether Sonar reports the issue on new code.

If using SonarCloud or a SonarQube edition that supports PR decoration:
- wire the PR integration in the next CI lessons.

---

## Quick Check

1. Why is PR analysis high ROI?
2. What does PR decoration mean?
3. Why might branch/PR features differ between SonarQube setups?

---

## Next

Go to **Lesson 10**: [GitHub Actions integration](10-ci-github-actions.md).
