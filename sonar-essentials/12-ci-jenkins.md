---
render_with_liquid: false
---
# 12 — CI Integration: Jenkins Pipeline (SonarQube)

Jenkins integrates well with SonarQube using the SonarQube Scanner and quality gate steps.

---

## 🎯 Learning Objectives

- Configure Jenkins credentials for Sonar tokens
- Run Sonar analysis from a Jenkinsfile
- Enforce Quality Gates as a pipeline stage

---

## 1) Typical Jenkins Setup

Common building blocks:
- SonarQube server configured in Jenkins (global config)
- Jenkins credential for `SONAR_TOKEN`
- A pipeline that:
  - builds/tests
  - runs analysis
  - waits for the quality gate

---

## 2) Example Jenkinsfile (Conceptual)

```groovy
pipeline {
  agent any

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test') {
      steps {
        sh 'mvn -B test'
      }
    }

    stage('Sonar Analysis') {
      steps {
        withSonarQubeEnv('sonarqube-server') {
          sh 'mvn -B sonar:sonar'
        }
      }
    }

    stage('Quality Gate') {
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
  }
}
```

Notes:
- `withSonarQubeEnv(...)` and `waitForQualityGate()` require Jenkins SonarQube integration.
- Adjust build/test commands for Gradle, Node, .NET, etc.

---

## 3) Operational Tips

- Always run tests before analysis so coverage can be imported.
- Make sure Jenkins uses full Git history when you need blame/trends.
- If quality gate checks slow pipelines, keep timeouts and investigate server performance.

---

## ✅ Hands-on Lab

1. Add SonarQube server configuration in Jenkins.
2. Add `SONAR_TOKEN` as a Jenkins credential.
3. Create a pipeline with the 4 stages.
4. Intentionally introduce a new bug/vulnerability and confirm the gate blocks the merge/deploy.

---

## Quick Check

1. Why do we run tests before the Sonar analysis stage?
2. What should happen when a quality gate fails?
3. What’s the role of the Jenkins SonarQube plugin integration?

---

## Next

Go to **Lesson 13**: [Language-specific scanners](13-language-integrations.md).
