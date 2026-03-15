---
render_with_liquid: false
---
# 10 — CI Integration: GitHub Actions (SonarQube + SonarCloud)

GitHub Actions is one of the most common places to run Sonar analysis.

Goal: every push/PR triggers analysis and produces a pass/fail signal.

---

## 🎯 Learning Objectives

- Add Sonar analysis to a GitHub Actions workflow
- Store tokens safely in GitHub Secrets
- Run analysis for PRs and main branch
- Understand options for enforcing Quality Gates

---

## 1) Required Secrets

In your GitHub repo settings → **Secrets and variables**:

- `SONAR_TOKEN`: Sonar token
- `SONAR_HOST_URL`: Only for SonarQube (example: `https://sonarqube.company.com`)

For SonarCloud, you often don’t need `SONAR_HOST_URL` (host is implicit), but keep your workflow flexible.

---

## 2) Minimal Workflow (Generic)

This example uses `sonar-scanner` (works for many repos but you must configure sources/tests/coverage correctly).

```yaml
name: Build and Sonar

on:
  push:
    branches: [ main ]
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # Run your tests first (examples)
      # - run: npm ci && npm test
      # - run: mvn -B test
      # - run: dotnet test

      - name: Sonar scan
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
        run: |
          sonar-scanner \
            -Dsonar.login=$SONAR_TOKEN \
            ${SONAR_HOST_URL:+-Dsonar.host.url=$SONAR_HOST_URL}
```

Notes:
- `fetch-depth: 0` helps with blame/branch analysis.
- For PR decoration you typically need the right integration + scanner parameters.

---

## 3) Enforcing the Quality Gate

There are two common approaches:

### Approach A: Fail on gate (preferred)
- Run analysis
- Wait for server to compute the gate
- Fail the job if gate fails

Depending on your scanner and platform, you can:
- use a dedicated “quality gate” step/action
- or configure the scanner to wait for gate results (where supported)

### Approach B: Report-only
- Run analysis, but don’t fail the pipeline
- Useful in early adoption

---

## ✅ Hands-on Lab

1. Add a GitHub Actions workflow to a repo.
2. Store `SONAR_TOKEN` in GitHub Secrets.
3. Run a workflow on a PR.
4. Verify you can see:
   - analysis on the PR
   - project updated on merge to `main`

---

## Quick Check

1. Why is `fetch-depth: 0` often recommended?
2. Where should `SONAR_TOKEN` be stored?
3. What’s the difference between “report-only” and “enforced” gates?

---

## Next

Go to **Lesson 11**: [GitLab CI integration](11-ci-gitlab.md).
