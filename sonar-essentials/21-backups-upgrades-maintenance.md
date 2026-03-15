---
render_with_liquid: false
---
# 21 — Backups, Upgrades, and Maintenance (Operate SonarQube Safely)

If you self-host SonarQube, you own:
- backups
- upgrades
- reliability

This lesson gives you a pragmatic ops approach.

---

## 🎯 Learning Objectives

- Identify what must be backed up
- Plan safe upgrades (especially across major versions)
- Understand common maintenance tasks

---

## 1) What to Back Up

In most deployments, critical data includes:
- **Database** (projects, issues, history, users)
- **SonarQube data directories/volumes** (varies by deployment)
- Configuration (reverse proxy, env vars, secrets)

If you run SonarQube via Docker:
- Back up Postgres volume
- Back up SonarQube volumes (data/extensions/logs) as appropriate

---

## 2) Upgrade Strategy

Safe upgrade rules:
- Prefer **LTS** versions for production
- Read release notes for breaking changes
- Test upgrades in staging first
- Keep DB backups before upgrade

Practical flow:
1. Backup DB + relevant volumes
2. Stop SonarQube
3. Upgrade image/version
4. Start SonarQube and let it migrate
5. Verify health + sample analyses

---

## 3) Plugin & Compatibility Notes

If you rely on plugins:
- confirm plugin compatibility before upgrade
- keep a list of installed plugins and versions

---

## 4) Maintenance Tasks

- Monitor disk usage (search indices can grow)
- Monitor DB size
- Keep an eye on compute queue (backlog indicates performance issues)
- Rotate logs if needed

---

## ✅ Hands-on Lab

1. Document your backup plan:
   - what you back up
   - where it’s stored
   - restore procedure
2. Simulate a “restore drill” (even just as a checklist).
3. Plan an LTS upgrade path (staging → production).

---

## Quick Check

1. What is the most critical backup component?
2. Why test upgrades in staging?
3. What’s a red flag that SonarQube performance is degrading?

---

## Next

Go to **Lesson 22**: performance and scaling basics.
