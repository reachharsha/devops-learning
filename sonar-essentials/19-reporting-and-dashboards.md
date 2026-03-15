---
render_with_liquid: false
---
# 19 — Reporting & Dashboards (What Leaders and Teams Actually Need)

Dashboards should drive action, not vanity metrics.

This lesson shows how to build reporting that helps:
- developers
- engineering managers
- security teams

---

## 🎯 Learning Objectives

- Choose a small set of meaningful metrics
- Build a weekly reporting rhythm
- Track trends without punishing teams for legacy debt

---

## 1) The Core Metrics That Matter

For most teams, these drive outcomes:
- New vulnerabilities (trend + count)
- New bugs (trend + count)
- Quality Gate pass rate
- Coverage on new code
- Time-to-fix for high severity issues

Avoid:
- obsessing over total issue count in large legacy codebases

---

## 2) Weekly Reporting Rhythm

A good weekly rhythm:
- identify top failing projects (by gate)
- review high-severity new issues
- pick one legacy hotspot to improve

This keeps quality moving without derailing delivery.

---

## 3) Team Dashboards

A useful team dashboard answers:
- which repos are failing gates?
- which repos have new vulnerabilities?
- what’s trending worse over the last 30 days?

---

## 4) Executive View (Keep it Small)

Executives don’t need “line count” charts.

They need:
- risk trend
- gate compliance trend
- where investment is needed (hotspots)

---

## ✅ Hands-on Lab

1. Pick 3 repos and define “success” for each.
2. Write a 5-line weekly report template:
   - Gate pass rate
   - New vulnerabilities
   - New bugs
   - Coverage on new code
   - Focus area this week

---

## Quick Check

1. Why is total issue count a poor KPI for legacy code?
2. What are 3 metrics you would track weekly?
3. What does a good executive report focus on?

---

## Next

Go to **Lesson 20**: SonarQube administration.
