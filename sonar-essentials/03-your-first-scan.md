---
render_with_liquid: false
---
# 03 — Your First Scan (Local + CI-Ready)

In this lesson you’ll run your first Sonar analysis and learn what’s actually happening.

---

## 🎯 Learning Objectives

- Understand the “scanner → server → report” flow
- Run a scan using the CLI scanner (generic)
- Know when to use Maven/Gradle/.NET scanners instead
- Read the first dashboard results confidently

---

## The Scan Flow (Mental Model)

1. **Scanner** runs in your repo
2. It parses code + reads config
3. It uploads analysis to SonarQube/SonarCloud
4. Server computes metrics/issues
5. You review results + quality gate

---

## Choose the Right Scanner

Use the most native scanner you can:

- **Maven**: `mvn sonar:sonar`
- **Gradle**: `./gradlew sonarqube`
- **.NET**: `dotnet sonarscanner begin/end`
- **Generic**: `sonar-scanner` (works but requires more manual config)

For learning and mixed stacks, we’ll start with `sonar-scanner`.

---

## Step 1 — Create a Project in SonarQube

In SonarQube UI:
1. Create a new project
2. Choose “manually”
3. Note the **Project Key**
4. Generate a **token**

Keep the token secret.

---

## Step 2 — Add a Minimal `sonar-project.properties`

At your repo root, create:

```properties
sonar.projectKey=my-sample-project
sonar.projectName=My Sample Project
sonar.sources=.
sonar.sourceEncoding=UTF-8

# optional: limit scope
# sonar.exclusions=**/node_modules/**,**/dist/**,**/build/**
```

---

## Step 3 — Run the Scan

### If using SonarQube locally

```bash
sonar-scanner \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN
```

### If your project is Java + Maven (recommended for Java)

If you already have a Maven build, prefer the Maven scanner:

```bash
mvn -B test
mvn -B sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=YOUR_TOKEN
```

### If using SonarCloud
You’ll typically use CI + the SonarCloud action/pipe, but local runs can work if configured.

---

## Step 4 — Review Results

Open your project in Sonar UI and look for:
- Issues by type (Bug, Vulnerability, Code Smell)
- Code smells count and examples
- Duplications
- Quality Gate status

### Don’t panic on first run
A legacy repo can show many findings. That’s expected.

Focus on:
- New code policy
- Biggest/most severe issues
- Establishing a workflow

---

## ✅ Hands-on Lab

1. Create a new project in SonarQube.
2. Add `sonar-project.properties`.
3. Run a scan.
4. Pick 2 issues and:
   - read the rule description
   - locate the code
   - decide whether it’s real, false positive, or needs review

---

## Quick Check

1. Where does the token belong?
2. What is the difference between native scanners and the generic scanner?
3. What should you focus on first: all legacy issues or new code?

---

## Next

Go to **Lesson 04**: Project configuration (keys, tokens, properties).
