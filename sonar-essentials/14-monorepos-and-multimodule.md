---
render_with_liquid: false
---
# 14 — Monorepos & Multi-Module Projects (Patterns That Work)

Monorepos and multi-module builds are where Sonar setups often fail.

This lesson focuses on **simple, maintainable patterns**.

---

## 🎯 Learning Objectives

- Choose between “one Sonar project” vs “many Sonar projects”
- Run analysis per module safely
- Keep ownership clear
- Avoid giant, noisy dashboards

---

## 1) Pattern A: One Sonar Project for the Whole Repo

Pros:
- simplest governance
- one dashboard

Cons:
- noisy if unrelated systems live together
- harder to assign ownership

Works best when:
- repo is one product with multiple packages


## 2) Pattern B: One Sonar Project per Module/Service

Pros:
- clearer ownership
- better signal per team/service

Cons:
- more configuration
- more CI steps

Works best when:
- repo contains multiple microservices owned by different teams

---

## 3) Practical Execution: Scan Subdirectories

A maintainable approach is multiple scans:
- each scan sets a different `sonar.projectKey`
- each scan sets `sonar.projectBaseDir`

Conceptually:

```bash
# scan service-a
( cd services/service-a && sonar-scanner -Dsonar.projectKey=org_service-a -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL )

# scan service-b
( cd services/service-b && sonar-scanner -Dsonar.projectKey=org_service-b -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL )
```

---

## 4) Keep Exclusions Correct

Monorepos often include:
- generated clients
- vendor libs
- build outputs

Be strict with exclusions so you analyze *source code*, not artifacts.

---

## ✅ Hands-on Lab

1. Pick a monorepo or create a simple one with two modules.
2. Implement Pattern A or Pattern B.
3. Ensure:
   - each project has correct source counts
   - coverage paths match
   - dashboards are meaningful

---

## Quick Check

1. When is “one Sonar project per module” the better choice?
2. Why do monorepos often produce noisy results?
3. What does `sonar.projectBaseDir` conceptually control?

---

## Next

Go to **Lesson 15**: Vulnerabilities vs Security Hotspots.
