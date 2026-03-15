---
render_with_liquid: false
---
# 04 — Project Configuration: Keys, Tokens, and Analysis Properties

If Sonar is misconfigured, it becomes noisy, slow, and unreliable.

This lesson teaches you how to configure analysis in a way that survives real CI/CD.

---

## 🎯 Learning Objectives

- Understand project keys, organization keys (SonarCloud), and tokens
- Configure analysis using properties + CI environment variables
- Set sensible exclusions (generated code, dependencies)
- Handle multi-module and monorepo basics safely

---

## 1) Project Key vs Project Name

- **Project Key**: a stable identifier used by Sonar internally
- **Project Name**: human-friendly label in the UI

Best practice:
- Keep the project key stable
- Use a consistent naming scheme

Example key patterns:
- `org_service-api`
- `team1_payments`
- `platform_monorepo`

---

## 2) Tokens (Authentication)

Tokens are used by scanners to authenticate.

Rules:
- Store tokens in **CI secrets** (GitHub Secrets, GitLab CI vars, Jenkins credentials)
- Avoid storing tokens in files
- Rotate tokens periodically
- Prefer least privilege when possible

---

## 3) Where to Put Configuration

You have three common approaches:

### A) `sonar-project.properties` (repo file)
Good for:
- CLI scanning
- Polyglot repos
- Simple setups

### B) Build tool config (Maven/Gradle/.NET)
Good for:
- Java/.NET projects where build tool knows sources/tests

### C) CI-only config (env vars)
Good for:
- organizations that standardize pipelines

In practice, teams often mix A + C.

---

## 4) Common Properties (Practical Set)

```properties
sonar.projectKey=org_service-api
sonar.projectName=Service API
sonar.sources=src
sonar.tests=test
sonar.sourceEncoding=UTF-8

# Keep Sonar focused
sonar.exclusions=**/node_modules/**,**/dist/**,**/build/**,**/coverage/**

# If you generate code, exclude it
# sonar.exclusions=**/generated/**
```

### Avoid scanning dependencies
- `node_modules/`, `vendor/`, `target/`, `dist/`, `build/` should usually be excluded.

---

## 5) Monorepo Basics

For monorepos, decide one of:

### Option 1: One Sonar project per repo
- easier governance
- can be noisy if many unrelated modules

### Option 2: One Sonar project per module/package
- better ownership and signal
- more setup effort

Either is valid. The best choice is the one that your team will maintain.

---

## ✅ Hands-on Lab

1. Add a clean `sonar-project.properties`.
2. Exclude build outputs (`dist/`, `build/`, etc.).
3. Re-run a scan and confirm:
   - analysis is faster
   - fewer irrelevant files are counted

---

## Quick Check

1. Why should the project key be stable?
2. Where should tokens live?
3. Why exclude generated/build output directories?

---

## Next

Go to **Lesson 05**: [Issues, rules, and Quality Profiles](05-rules-and-quality-profiles.md).
