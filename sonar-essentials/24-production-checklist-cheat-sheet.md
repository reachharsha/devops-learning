---
render_with_liquid: false
---
# 24 — Production Checklist + Cheat Sheet (SonarQube/SonarCloud)

This final lesson summarizes the course into a practical checklist and a small command/reference section.

---

## ✅ Production Checklist (SonarQube)

### Server
- Use an appropriate edition for your needs
- Prefer LTS for production
- Run behind HTTPS (reverse proxy)
- Restrict admin access

### Database
- Use a supported DB (commonly Postgres)
- Back up DB regularly
- Test restore procedure

### Security
- Tokens stored in secret managers
- Rotate tokens periodically
- Monitor access logs (where possible)

### Configuration
- Standardize:
  - Quality Profiles per language
  - Quality Gates per repo tier
  - “new code” definition

### CI/CD
- Sonar runs on PRs and main
- Gates enforced (or a clear timeline to enforce)
- Tests run before analysis
- Coverage is imported correctly

### Observability
- Monitor disk usage
- Monitor compute queue
- Monitor DB health

---

## ✅ Production Checklist (SonarCloud)

- Org and repos imported correctly
- Tokens stored in CI secrets
- PR analysis enabled
- Gates enforced on new code
- Reporting rhythm established (weekly)

---

## 🧾 Cheat Sheet

### Local SonarQube (Docker)

```bash
# start
docker compose up -d

# status
docker compose ps

# logs
docker compose logs -f sonarqube

# stop
docker compose down
```

### Generic scanner (example)

```bash
sonar-scanner \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=$SONAR_TOKEN
```

---

## 🎓 What to Do Next

Pick one real repo and implement the full workflow:
1. Correct configuration + exclusions
2. Tests + coverage import
3. New-code quality gate
4. PR analysis and enforcement
5. Weekly reporting rhythm

---

## Congratulations

If you completed the labs, you now have a complete Sonar adoption playbook.

You can confidently:
- add Sonar to CI
- tune rules
- enforce gates
- operate SonarQube
- build a sustainable triage workflow
