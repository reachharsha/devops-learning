---
render_with_liquid: false
---
# 13 — Language Integrations (Java, JS/TS, .NET, Python)

Sonar is most reliable when you use the **best scanner for your ecosystem**.

---

## 🎯 Learning Objectives

- Choose the right scanner per language
- Understand what information scanners can auto-detect
- Know the minimum “working” command per stack
- Wire coverage import at a high level

---

## 1) Java (Maven)

Typical command:

```bash
mvn -B test
mvn -B sonar:sonar -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL
```

Why Maven scanner is preferred:
- It knows source/test directories
- It understands multi-module Maven projects
- It integrates well with CI


## 2) Java (Gradle)

```bash
./gradlew test
./gradlew sonarqube -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL
```


## 3) JavaScript / TypeScript (Node)

Common approach:
1. Run `npm test` with coverage enabled
2. Provide `lcov.info` path to Sonar

```bash
npm ci
npm test
sonar-scanner -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL
```


## 4) .NET

The .NET scanner typically uses a “begin/build/end” flow.

Conceptually:
```bash
dotnet sonarscanner begin /k:"mykey" /d:sonar.login="$SONAR_TOKEN" /d:sonar.host.url="$SONAR_HOST_URL"
dotnet build
dotnet test
dotnet sonarscanner end /d:sonar.login="$SONAR_TOKEN"
```


## 5) Python

Typical flow:
```bash
pytest
# generate coverage.xml
sonar-scanner -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL
```


## 6) Polyglot Repos

If your repo contains multiple languages:
- you can often use the CLI scanner with a good `sonar-project.properties`
- but ensure you correctly define sources/tests and exclusions per language

---

## ✅ Hands-on Lab

Pick your stack and:
1. Switch from the generic scanner to the native scanner (if available).
2. Confirm analysis results are stable and accurate.
3. Add coverage import and verify it shows in Sonar.

---

## Quick Check

1. Why do native scanners usually produce better results?
2. What’s the common failure mode when scanning JS/TS repos?
3. What does the .NET “begin/build/end” pattern achieve?

---

## Next

Go to **Lesson 14**: Monorepos & multi-module projects.
