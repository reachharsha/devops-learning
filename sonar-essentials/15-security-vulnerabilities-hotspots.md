---
render_with_liquid: false
---
# 15 — Security in Sonar: Vulnerabilities vs Security Hotspots

Security is one of the highest-value reasons to adopt Sonar.

But teams often misread the results. This lesson teaches a safe and practical interpretation.

---

## 🎯 Learning Objectives

- Distinguish **Vulnerabilities** from **Security Hotspots**
- Build a review workflow that doesn’t overwhelm the team
- Use gates to prevent security regressions

---

## 1) Vulnerabilities

A **vulnerability** is a finding Sonar believes is a security flaw.

Examples (conceptual):
- Injection patterns
- Insecure randomness
- Hard-coded credentials
- Unsafe deserialization

How to treat them:
- Prioritize by exploitability and exposure
- Fix quickly on new code
- Add tests to prevent regression

### Java examples (typical vulnerability patterns)

#### Example A: SQL injection (vulnerability)

```java
public List<User> search(Connection conn, String q) throws Exception {
   String sql = "SELECT id, name FROM users WHERE name LIKE '%" + q + "%'";
   try (Statement st = conn.createStatement();
       ResultSet rs = st.executeQuery(sql)) {
      List<User> out = new ArrayList<>();
      while (rs.next()) out.add(new User(rs.getLong(1), rs.getString(2)));
      return out;
   }
}
```

Fix: use parameterized queries (PreparedStatement) and escape wildcard patterns appropriately.

#### Example B: Hard-coded credential (vulnerability)

```java
private static final String DB_PASSWORD = "P@ssw0rd";
```

Fix: move secrets to environment variables / secret manager, rotate immediately.

---

## 2) Security Hotspots

A **security hotspot** is not necessarily a vulnerability.

It means:
- “This code is sensitive; a human must review it.”

Common hotspot patterns:
- Building SQL queries dynamically
- Handling credentials
- Using cryptography APIs
- Opening redirects

Hotspots require:
- a reviewer
- context (where input comes from, what controls exist)

### Java examples (typical hotspot patterns)

#### Example C: Crypto configuration (hotspot)

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, key);
```

Why hotspot?
- Crypto is easy to misuse.
- ECB mode is often unsafe; a reviewer should confirm the threat model and recommend safer modes.

#### Example D: Deserialization or parsing untrusted input (hotspot)

```java
ObjectInputStream ois = new ObjectInputStream(request.getInputStream());
Object obj = ois.readObject();
```

Why hotspot?
- If input is attacker-controlled, unsafe deserialization can lead to RCE in some environments.
- Requires careful review: is input trusted, is there validation, can format be replaced?

---

## 3) How to Build a Review Workflow

A practical workflow:
1. Gate on **new vulnerabilities** = 0
2. Track hotspots, but don’t block delivery on legacy hotspots initially
3. Require hotspot review for new code
4. Add a security owner rotation to review hotspots weekly

---

## 4) Security Gates (Practical Starter)

Recommended initial security conditions on New Code:
- New vulnerabilities: **0**
- New security hotspots: reviewed (process-based)

Avoid:
- Blocking on “all hotspots ever” in a legacy system

---

## ✅ Hands-on Lab

1. In your project, locate:
   - 1 vulnerability (if present)
   - 1 hotspot (if present)
2. For each, answer:
   - What is the threat scenario?
   - Is the input attacker-controlled?
   - What is the impact?
   - What is the smallest safe fix?
3. Add a Quality Gate rule: New Vulnerabilities = 0

---

## Quick Check

1. Why aren’t hotspots always vulnerabilities?
2. What’s a safe way to enforce security without blocking teams permanently?
3. What should the default target for new vulnerabilities be?

---

## Next

Go to **Lesson 16**: Triage workflow.
