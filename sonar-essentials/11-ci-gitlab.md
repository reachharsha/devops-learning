---
render_with_liquid: false
---
# 11 — CI Integration: GitLab CI (SonarQube + SonarCloud)

GitLab CI is a strong match for Sonar because it has first-class pipeline variables and merge request workflows.

---

## 🎯 Learning Objectives

- Add Sonar analysis to `.gitlab-ci.yml`
- Store tokens safely in GitLab CI/CD variables
- Cache scanner files to speed up pipelines
- Enforce Quality Gates

---

## 1) Store Secrets in GitLab

Project → Settings → CI/CD → Variables:
- `SONAR_TOKEN` (masked)
- `SONAR_HOST_URL` (only for SonarQube)

---

## 2) Example `.gitlab-ci.yml`

```yaml
stages:
  - test
  - sonar

test:
  stage: test
  image: node:20
  script:
    - npm ci
    - npm test
  artifacts:
    when: always
    paths:
      - coverage/

sonar:
  stage: sonar
  image:
    name: sonarsource/sonar-scanner-cli:latest
    entrypoint: [""]
  variables:
    SONAR_USER_HOME: "$CI_PROJECT_DIR/.sonar"
    GIT_DEPTH: "0"
  cache:
    key: "$CI_JOB_NAME"
    paths:
      - .sonar/cache
  script:
    - sonar-scanner -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL
  only:
    - merge_requests
    - main
```

Notes:
- Adapt the `test` job to your stack.
- Ensure your repo includes `sonar-project.properties` or build-tool config.

---

## 3) Quality Gate Enforcement

Ways to enforce:
- Use a dedicated gate check job/step
- Or configure the scanner to wait for the quality gate (where supported)

During early rollout, you can run Sonar on MRs with **soft enforcement**:
- collect results
- educate teams
- flip to strict enforcement once stable

---

## ✅ Hands-on Lab

1. Add `.gitlab-ci.yml` with test + sonar stages.
2. Create CI variables for token and host.
3. Open a merge request and confirm Sonar analysis runs.
4. Tune your Quality Gate to “new code” and observe pass/fail behavior.

---

## Quick Check

1. Why do we set `GIT_DEPTH: 0`?
2. Why cache `.sonar/cache`?
3. What’s a safe rollout strategy for quality gates?

---

## Next

Go to **Lesson 12**: [Jenkins integration](12-ci-jenkins.md).
