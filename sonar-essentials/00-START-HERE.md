---
render_with_liquid: false
---
# 🔎 Sonar Essentials (SonarQube + SonarCloud) — Complete Learning Path

> **From zero to production-grade code quality & security in CI/CD**

This course teaches you how to use **SonarQube** (self-hosted) and **SonarCloud** (hosted) to continuously measure and improve:
- Code quality (bugs, code smells, maintainability)
- Security (vulnerabilities, security hotspots)
- Reliability, readability, duplication
- Test coverage (when paired with your test toolchain)

You’ll learn how to **run scans locally**, **integrate with CI**, set **quality gates**, and operate Sonar safely in real teams.

---

## 📚 Course Overview

- **Total Lessons:** 24
- **Estimated Time:** 6–8 weeks (6–10 hours/week)
- **Level:** Beginner → Advanced
- **Prerequisites:** Git basics + ability to run a build/test command for at least one language (Java/JS/.NET/Python)

---

## 🎯 Outcomes (What you’ll be able to do)

By the end, you will be able to:

- Explain what Sonar measures and what it doesn’t
- Stand up **SonarQube with Docker** (recommended dev setup)
- Scan a project with the right **scanner** (Maven/Gradle/.NET/CLI)
- Configure project analysis (`sonar-project.properties`, CI env vars)
- Use **Quality Profiles** and **Quality Gates** effectively
- Add **Pull Request analysis** and **branch analysis**
- Import test/coverage reports correctly
- Build a triage workflow: **new code first**, then legacy
- Operate SonarQube: users, permissions, backups, upgrades
- Troubleshoot scanner and server issues quickly

---

## 🧭 Choose Your Path: SonarQube vs SonarCloud

**SonarQube (self-hosted)**
- You manage the server (DB, upgrades, backups)
- Best when you need on-prem / private network / strict control

**SonarCloud (hosted)**
- No server ops; faster start
- Great for GitHub/GitLab projects (especially open-source)

Throughout the course, lessons show **both** when they differ.

---

## ✅ Prerequisites Checklist

Install/confirm:
- Docker Desktop (for local SonarQube) 
- Git
- A build tool for your stack (one of):
  - Java: Maven or Gradle
  - JavaScript/TypeScript: Node.js + npm
  - .NET: `dotnet`
  - Python: `python` + `pytest`

Nice-to-have:
- A sample repo you can modify freely

---

## 🗺️ Lesson Map

### Phase 1 — Foundations (00–04)
- **00**: Start here (this page)
- **01**: What is Sonar? (concepts + vocabulary)
- **02**: Install SonarQube (Docker) + SonarCloud setup
- **03**: Your first scan (local + CI-ready)
- **04**: Project configuration (keys, tokens, properties)

### Phase 2 — Quality Workflow (05–09)
- **05**: Issues, rules, and Quality Profiles
- **06**: Metrics: bugs, smells, duplication, maintainability
- **07**: Quality Gates (policy you can enforce)
- **08**: Tests + coverage (what Sonar reads, what it doesn’t)
- **09**: Branches + Pull Requests (new code workflow)

### Phase 3 — CI Integration (10–14)
- **10**: CI with GitHub Actions
- **11**: CI with GitLab CI
- **12**: CI with Jenkins
- **13**: Language integrations (Java, JS/TS, .NET, Python)
- **14**: Monorepos & multi-module projects

### Phase 4 — Security & Triage (15–19)
- **15**: Vulnerabilities vs Security Hotspots
- **16**: Triage workflow (new code first) + debt strategy
- **17**: False positives, suppression, and rule tuning
- **18**: Governance: gate design for teams
- **19**: Reporting & dashboards that matter

### Phase 5 — Operations & Production (20–24)
- **20**: SonarQube administration (users, permissions)
- **21**: Backups, upgrades, and maintenance
- **22**: Performance and scaling basics
- **23**: Troubleshooting playbook
- **24**: Production checklist + cheat sheet

---

## 🧪 How to Use This Course

Recommended loop per lesson:
1. Read the lesson once quickly
2. Do the hands-on lab
3. Answer the “Quick Check” questions
4. Implement the same step in your own repo

---

## 🧰 The Core Mental Model

Sonar is best used as:
- A **policy engine** (quality gates on new code)
- A **feedback loop** (PR annotations + trends)
- A **triage system** (prioritize what matters)

Sonar is not:
- A replacement for code review
- A replacement for tests
- A generic “security scanner for everything”

---

## Next

Start with **Lesson 01**: [What is Sonar?](01-what-is-sonar.md)
