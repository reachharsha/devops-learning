---
render_with_liquid: false
---
# 22 — Performance & Scaling (Keep Analysis Fast and Reliable)

SonarQube has multiple internal components, and performance problems usually show up as:
- slow analysis feedback
- stuck compute tasks
- timeouts in CI

This lesson focuses on the basics you need to keep it healthy.

---

## 🎯 Learning Objectives

- Understand SonarQube’s high-level components
- Recognize common performance bottlenecks
- Apply practical scaling patterns

---

## 1) High-Level Components (Conceptual)

Most SonarQube deployments include:
- Web/UI + API
- Compute engine (processes analysis tasks)
- Search/indexing
- Database

If compute tasks pile up, quality gate feedback gets slow.

---

## 2) Common Bottlenecks

### CPU/RAM
- insufficient memory can cause instability

### Disk IO
- DB and search/indexing require decent IO

### Database
- slow DB = slow everything

### Too much noise in analysis
- scanning `node_modules` / build outputs makes analysis slow and useless

---

## 3) Practical Scaling Patterns

- Ensure exclusions are correct (biggest win)
- Use stronger DB resources for production
- Keep SonarQube on stable LTS
- Separate DB storage from container ephemeral storage

---

## ✅ Hands-on Lab

1. Measure analysis time for your repo.
2. Tighten exclusions and measure again.
3. Create a small “performance checklist” you will apply to every new repo.

---

## Quick Check

1. What’s the fastest way to speed up analysis in a noisy repo?
2. Why does DB performance matter so much?
3. What does a growing compute queue indicate?

---

## Next

Go to **Lesson 23**: [troubleshooting playbook](23-troubleshooting-playbook.md).
