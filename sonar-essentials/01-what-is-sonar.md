---
render_with_liquid: false
---
# 01 — What Is Sonar? (SonarQube & SonarCloud)

Sonar (from SonarSource) is a platform for **continuous code quality and security analysis**.

It answers questions teams repeatedly struggle with:
- Are we introducing **new bugs**?
- Is the code getting harder to maintain?
- Did our PR introduce **security issues**?
- Are we accumulating technical debt faster than we’re paying it down?

---

## 🎯 Learning Objectives

By the end of this lesson, you will be able to:
- Explain what Sonar analyzes (and what it doesn’t)
- Understand the main Sonar concepts: **rules, issues, profiles, gates**
- Read the Sonar dashboard without confusion
- Adopt the correct “new code first” mindset

---

## 🧠 Sonar’s Core Idea: Feedback on Every Change

A healthy workflow is:
1. Developer pushes a branch / opens a PR
2. CI runs tests + builds
3. CI runs **Sonar analysis**
4. Sonar reports issues *on new/changed code*
5. Quality Gate passes/fails (policy)

This shifts quality from “we’ll clean it later” to **continuous hygiene**.

---

## 📌 Key Vocabulary

### 1) Rule
A rule describes a pattern in code that may be:
- a bug (reliability)
- a code smell (maintainability)
- a vulnerability (security)
- a security hotspot (needs review)

### 2) Issue
An issue is a **specific finding** in your code produced by a rule.

### 3) Quality Profile
A Quality Profile is a **collection of enabled rules** (and their parameters) applied to projects.

You might have:
- A strict profile for production services
- A more permissive profile for experimental repos

### 4) Quality Gate
A Quality Gate is a **pass/fail policy** based on metrics (especially on **New Code**).

Example (simple, practical gate):
- New code has **0 new vulnerabilities**
- New code has **0 new bugs**
- Coverage on new code ≥ **80%**

---

## 📊 What Sonar Measures

You will commonly see:
- **Bugs**: likely coding mistakes
- **Vulnerabilities**: security flaws (e.g., injection patterns)
- **Security Hotspots**: suspicious patterns requiring human review
- **Code Smells**: maintainability issues
- **Duplications**: repeated code blocks
- **Coverage**: based on test/coverage reports produced by your tools

### What Sonar does NOT do by itself
- It does not run your tests.
- It does not create coverage reports.
- It does not magically know runtime behavior.

Sonar consumes outputs from:
- Your test runner (JUnit, pytest, Jest)
- Coverage tools (JaCoCo, cobertura, lcov, dotnet coverage)

---

## 🧩 SonarQube vs SonarCloud in One Sentence

- **SonarQube**: you run a server; best for internal/private environments.
- **SonarCloud**: hosted; best for quick onboarding and tight VCS integration.

Both use the same mental model: analyze → report → gate.

---

## ✅ “New Code First” (The One Habit That Makes Sonar Work)

If you enable Sonar on a legacy codebase, it may produce thousands of issues.

The winning strategy:
- Use a Quality Gate focused on **New Code**
- Fix what you touch
- Gradually reduce legacy debt

This prevents “analysis fatigue” and makes Sonar sustainable.

---

## 🧪 Hands-on Preview (No Install Yet)

In later lessons you’ll run a scan. For now, just confirm:
- You can build and test one project locally.

Pick a repo and run one:
- Java: `mvn test` or `./gradlew test`
- Node: `npm test`
- .NET: `dotnet test`
- Python: `pytest`

---

## Quick Check

1. What is the difference between a **Quality Profile** and a **Quality Gate**?
2. Why is “new code first” the recommended adoption strategy?
3. Does Sonar generate coverage by itself?

---

## Next

Go to **Lesson 02**: [Install SonarQube (Docker) + SonarCloud setup](02-install-sonarqube.md).
