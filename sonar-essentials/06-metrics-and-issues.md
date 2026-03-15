---
render_with_liquid: false
---
# 06 — Metrics & Issues: Reading Sonar Like a Pro

A Sonar dashboard can look intimidating at first. This lesson teaches you what matters and what to ignore.

---

## 🎯 Learning Objectives

- Interpret the main metrics (reliability, security, maintainability)
- Understand issue categories and severities
- Know how to prioritize fixes
- Avoid “metric theater” (numbers without outcomes)

---

## 1) Issue Types (The Big Four)

### Bugs (Reliability)
Likely to cause incorrect behavior (or crash).

### Vulnerabilities (Security)
Patterns that can be exploited.

### Security Hotspots
Potentially risky patterns that require **human review**.

### Code Smells (Maintainability)
Code that is harder to change safely (complexity, duplication, unclear logic).

---

## Java Examples (What Each Type Looks Like)

These are small, realistic snippets similar to what Sonar rules often flag.

### A) Bug (Reliability): Possible NullPointerException

**Problem:** `user` can be null → runtime crash.

```java
public String displayName(User user) {
   return user.getFirstName() + " " + user.getLastName();
}
```

**Safer:**

```java
public String displayName(User user) {
   if (user == null) return "(unknown)";
   return user.getFirstName() + " " + user.getLastName();
}
```

### B) Code Smell (Maintainability): Overly complex branching

**Problem:** deeply nested conditionals raise complexity and make changes risky.

```java
public int calculatePrice(Order order) {
   int price = 0;
   if (order != null) {
      if (order.isVip()) {
         if (order.getItems() != null) {
            if (order.getItems().size() > 10) {
               price = 100;
            } else {
               price = 120;
            }
         }
      } else {
         price = 150;
      }
   }
   return price;
}
```

**Safer/cleaner:**

```java
public int calculatePrice(Order order) {
   if (order == null) return 0;
   if (!order.isVip()) return 150;
   if (order.getItems() == null) return 120;
   return order.getItems().size() > 10 ? 100 : 120;
}
```

### C) Vulnerability (Security): SQL injection via string concatenation

**Problem:** attacker-controlled input becomes part of SQL.

```java
public User findUser(Connection conn, String username) throws Exception {
   String sql = "SELECT id, name FROM users WHERE username='" + username + "'";
   try (Statement st = conn.createStatement();
       ResultSet rs = st.executeQuery(sql)) {
      return rs.next() ? new User(rs.getLong("id"), rs.getString("name")) : null;
   }
}
```

**Safer:**

```java
public User findUser(Connection conn, String username) throws Exception {
   String sql = "SELECT id, name FROM users WHERE username = ?";
   try (PreparedStatement ps = conn.prepareStatement(sql)) {
      ps.setString(1, username);
      try (ResultSet rs = ps.executeQuery()) {
         return rs.next() ? new User(rs.getLong("id"), rs.getString("name")) : null;
      }
   }
}
```

### D) Security Hotspot: Crypto usage requires review

**What Sonar is telling you:** “This is security-sensitive; ensure it’s configured safely.”

```java
Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, secretKey);
byte[] encrypted = cipher.doFinal(plain);
```

Why hotspot?
- ECB mode is usually insecure for most data patterns.
- You should typically use an authenticated mode (e.g., GCM) with a random IV.

### E) Duplication (Maintainability): Same logic copied twice

```java
public boolean isEligibleForDiscount(Order order) {
   if (order == null) return false;
   if (order.getItems() == null) return false;
   return order.getItems().size() >= 5;
}

public boolean isEligibleForFreeShipping(Order order) {
   if (order == null) return false;
   if (order.getItems() == null) return false;
   return order.getItems().size() >= 5;
}
```

This is a typical duplication candidate: extract shared checks into one helper.

---

## 2) Severity vs Priority

Sonar reports severity (Blocker → Info), but your team should define priority.

A practical priority model:
- **P0**: exploitable vulnerability, auth bypass, secrets exposure
- **P1**: critical bug in a hot path, major reliability issues
- **P2**: maintainability issues that will slow the next refactor
- **P3**: minor stylistic smells (fix opportunistically)

---

## 3) Ratings and Technical Debt (Use Carefully)

Sonar often summarizes maintainability with:
- Ratings (A–E)
- Estimated “debt” time

These are helpful trends, but don’t treat them as exact accounting.

Use them for:
- comparing modules
- tracking trend over time
- deciding where to invest cleanup

Tip: the UI may show separate ratings for **Reliability**, **Security**, and **Maintainability** (names can vary between SonarQube versions/SonarCloud). Treat them as summaries of “how bad the current issues are” rather than absolute truth.

---

## 4) Duplications

Duplicated code increases:
- bug surface area
- cost of changes

But duplication isn’t always bad (some duplication is intentional to decouple).

Rule of thumb:
- Reduce duplication in shared logic
- Avoid premature “DRY” that harms readability

---

## 5) The Most Important Filter: New Code

If your repo has many issues, focus on:
- **New Code** issues
- Highest severity security/reliability

This keeps Sonar from becoming unmaintainable.

---

## ✅ Hands-on Lab: Triage Like a Team

1. Open your project dashboard.
2. Filter to **New Code**.
3. Pick 5 issues:
   - 2 security
   - 2 bugs
   - 1 maintainability
4. For each issue, write:
   - Why it matters
   - Whether it’s real or false positive
   - The smallest safe fix

---

## Quick Check

1. What’s the difference between a vulnerability and a security hotspot?
2. Why is “new code” the key adoption tool?
3. When can duplication be acceptable?

---

## Next

Go to **Lesson 07**: Quality Gates.
