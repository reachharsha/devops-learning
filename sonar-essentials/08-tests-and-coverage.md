---
render_with_liquid: false
---
# 08 — Tests & Coverage: What Sonar Reads (and How to Wire It Correctly)

Sonar displays **coverage**, but it doesn’t generate it.

Coverage comes from your tooling (JaCoCo, lcov, dotnet coverage, etc.) and is **imported** into Sonar.

---

## 🎯 Learning Objectives

- Understand how Sonar gets test and coverage data
- Import coverage for common stacks (Java, JS/TS, .NET, Python)
- Avoid the most common coverage pitfalls
- Use “coverage on new code” to drive good habits

---

## 1) The Coverage Pipeline

A reliable pipeline looks like:
1. Run tests
2. Generate coverage report file(s)
3. Run Sonar analysis and point it to the coverage report

If step 2 doesn’t happen, Sonar can’t show coverage.

---

## 2) Common Coverage Formats

- Java: JaCoCo XML
- JS/TS: lcov (`lcov.info`)
- .NET: OpenCover / VS coverage / other supported formats depending on setup
- Python: coverage.xml (Cobertura-style)

---

## 3) Practical Config Examples

### Java (JaCoCo)
Typical approach:
- Configure JaCoCo to produce XML
- Ensure it runs during CI
- Point Sonar to the report path

#### Java (Maven + JaCoCo) — Minimal working example

1) Add JaCoCo to `pom.xml` (generates XML at `target/site/jacoco/jacoco.xml`):

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.jacoco</groupId>
			<artifactId>jacoco-maven-plugin</artifactId>
			<version>0.8.12</version>
			<executions>
				<execution>
					<goals>
						<goal>prepare-agent</goal>
					</goals>
				</execution>
				<execution>
					<id>report</id>
					<phase>test</phase>
					<goals>
						<goal>report</goal>
					</goals>
					<configuration>
						<outputDirectory>${project.reporting.outputDirectory}/jacoco</outputDirectory>
					</configuration>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

2) Point Sonar to the XML report (either in `sonar-project.properties` or CI args):

```properties
sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
```

3) Run in CI (conceptually):

```bash
mvn -B test
mvn -B sonar:sonar -Dsonar.login=$SONAR_TOKEN -Dsonar.host.url=$SONAR_HOST_URL
```

If coverage shows 0%, the #1 cause is: Sonar can’t find the XML file or the paths don’t match.

### JavaScript/TypeScript (lcov)
Generate coverage with your test runner and export lcov.

### Python
Use `coverage.py` to produce XML.

---

## 4) The Biggest Pitfalls

### Pitfall A: Paths don’t match
Coverage report paths must match the source paths Sonar sees.

### Pitfall B: Coverage generated in a different working directory
CI often changes directories; keep paths consistent.

### Pitfall C: You’re scanning build output
Exclude `dist/`, `build/`, `coverage/` from sources.

### Pitfall D: Tests aren’t being run in CI
Sonar analysis without tests gives misleading confidence.

---

## ✅ Hands-on Lab

Pick your stack and do this end-to-end:
1. Run tests
2. Generate a coverage report file
3. Configure Sonar analysis to import it
4. Confirm the Sonar UI shows coverage

Bonus:
- Track “Coverage on New Code” and set a realistic target in your Quality Gate.

---

## Quick Check

1. Does Sonar run your tests?
2. What is the most common reason coverage shows as 0%?
3. Why is “coverage on new code” a good gate metric?

---

## Next

Go to **Lesson 09**: Branch and Pull Request analysis.
