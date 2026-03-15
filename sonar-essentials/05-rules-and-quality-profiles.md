---
render_with_liquid: false
---
# 05 — Rules & Quality Profiles (Make Sonar Match Your Standards)

Sonar becomes powerful when you treat it as **your organization’s coding standard** expressed as rules + profiles.

---

## 🎯 Learning Objectives

- Understand how rules create issues
- Use the built-in profiles safely
- Create and maintain a custom Quality Profile
- Tune noise: avoid disabling everything, avoid enabling everything

---

## 1) Rules: The Source of Findings

A rule includes:
- A description (why it’s a problem)
- Examples of “bad” vs “good” code
- A severity (Info/Minor/Major/Critical/Blocker)
- Sometimes parameters (thresholds)

Rules are usually grouped by:
- Bugs
- Vulnerabilities
- Code Smells
- Security Hotspots

---

## 2) Quality Profiles: Rule Sets Applied to Projects

A **Quality Profile** is a bundle of enabled rules.

Common patterns in teams:
- **Sonar way** (default) for general use
- **Team profile**: stricter subset aligned with your engineering guidelines
- **Legacy profile**: used temporarily to reduce noise while adopting “new code first”

### Best practice
Start with a sensible default profile, then tune gradually.

---

## 3) Strategies for Tuning Profiles

### Strategy A: Tighten on new code
- Keep legacy stable (don’t overwhelm the team)
- Make new code clean (raise the bar)

### Strategy B: Tune by language
A Java service and a JS frontend often need different rule tradeoffs.

### Strategy C: Enable security rules early
Security rules are usually higher ROI and less subjective than stylistic rules.

---

## 4) Avoid These Two Extremes

### “Disable everything noisy”
Result: Sonar becomes meaningless.

### “Enable everything strict”
Result: Sonar blocks delivery and gets turned off.

Aim for a profile that:
- catches real bugs and vulnerabilities
- flags maintainability issues that actually slow teams down
- stays stable (don’t change weekly)

---

## 5) Rule Parameters (Where Noise Usually Comes From)

Some rules have thresholds (e.g., complexity, duplicated lines, nesting depth).

Instead of disabling:
- adjust thresholds to fit your codebase maturity
- apply stricter thresholds only on new code via gates

---

## ✅ Hands-on Lab: Create a Team Profile

In Sonar UI:
1. Go to **Quality Profiles**
2. Pick your main language (e.g., Java / JavaScript)
3. Copy the default profile into “Team Profile”
4. Make 3 deliberate changes:
   - enable 1 security rule you want enforced
   - disable 1 rule you consider low value
   - tune 1 parameterized rule (threshold)
5. Apply the profile to your project
6. Re-run analysis and observe differences

---

## Quick Check

1. What’s the difference between a **rule** and an **issue**?
2. Why is gradual tuning better than big changes?
3. What’s the risk of “enable everything strict”?

---

## Next

Go to **Lesson 06**: [Metrics & issue types](06-metrics-and-issues.md).
