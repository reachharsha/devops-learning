---
render_with_liquid: false
---
# 23 — Troubleshooting Playbook (Scanner + Server)

When Sonar breaks, it usually breaks in repeatable ways.

This playbook helps you diagnose issues quickly.

---

## 🎯 Learning Objectives

- Debug common scanner failures
- Debug common SonarQube server failures
- Know the minimum set of logs to check

---

## 1) Scanner Failures

### A) Authentication errors (401/403)
Symptoms:
- “Not authorized”

Fix:
- confirm token is correct
- confirm token permissions
- ensure CI secrets are wired

### B) Cannot reach server
Symptoms:
- connection refused / timeout

Fix:
- verify `sonar.host.url`
- check network/firewall
- for local SonarQube, confirm container is up

### C) Project key mismatch
Symptoms:
- analysis goes to wrong project or fails

Fix:
- confirm `sonar.projectKey`
- confirm CI uses the correct repo config

---

## 2) Server Failures (SonarQube)

### A) UI loads but analysis never completes
Check:
- compute engine queue
- DB health

### B) Search/index issues
Symptoms:
- errors in logs
- dashboards missing data

Fix:
- check disk
- check memory
- restart after fixing resource issues (carefully)

---

## 3) What Logs to Check

Docker Compose (local):
```bash
docker compose logs -f sonarqube
```

Also check DB logs if there are connection errors.

---

## ✅ Hands-on Lab

1. Intentionally break your setup (pick one):
   - wrong token
   - wrong host URL
   - scan build output by removing exclusions
2. Diagnose using the steps above.
3. Fix and re-run analysis.

---

## Quick Check

1. What causes 401/403 errors most often?
2. What’s the first log you check in a local Docker SonarQube setup?
3. What indicates compute-engine backlog?

---

## Next

Go to **Lesson 24**: [production checklist + cheat sheet](24-production-checklist-cheat-sheet.md).
