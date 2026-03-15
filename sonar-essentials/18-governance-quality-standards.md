---
render_with_liquid: false
---
# 18 — Governance: Organization-Wide Standards That Teams Accept

Sonar governance is not about making everything strict.

It’s about:
- defining minimum standards
- enforcing them consistently
- leaving teams room to move fast safely

---

## 🎯 Learning Objectives

- Define sensible baseline profiles and gates
- Roll out Sonar across many repos without chaos
- Use “new code” gates to avoid blocking modernization

---

## 1) Establish Baselines

A practical baseline for most orgs:
- New vulnerabilities = 0
- New bugs = 0
- Coverage on new code: start at 60–80% depending on maturity
- Hotspots: reviewed on new code

---

## 2) Tiered Gates by Repo Type

Not all repos are equal.

Example tiers:
- Tier 1 (payments/auth): strict gates, higher coverage
- Tier 2 (core services): strong gates, medium coverage
- Tier 3 (internal tools): basic gates, low overhead

---

## 3) Rollout Strategy

Recommended rollout:
1. Enable Sonar in report-only mode
2. Tune exclusions and profiles (reduce noise)
3. Enforce new-code gates
4. Expand to PR decoration and strict gate enforcement

---

## 4) Ownership + Support Model

A governance model that works:
- platform team maintains the Sonar server/integration templates
- product teams own their project configuration and cleanup
- security team defines security rules + reviews hotspots

---

## ✅ Hands-on Lab

Design your org policy:
1. Define 1 baseline gate.
2. Define 2 tiers of stricter gates.
3. Choose a rollout timeline (e.g., 4 weeks) and write what changes each week.

---

## Quick Check

1. Why do tiered gates reduce resistance?
2. What’s the value of report-only mode?
3. Why does “new code first” make governance sustainable?

---

## Next

Go to **Lesson 19**: [reporting and dashboards](19-reporting-and-dashboards.md).
