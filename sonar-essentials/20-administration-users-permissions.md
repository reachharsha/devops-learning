---
render_with_liquid: false
---
# 20 — SonarQube Administration: Users, Permissions, and Safe Defaults

Running SonarQube in a team environment requires basic admin skills.

This lesson covers what to lock down and how to avoid common security mistakes.

---

## 🎯 Learning Objectives

- Understand roles, groups, and permissions
- Manage tokens safely
- Apply safe defaults for a team setup
- Know what to automate vs what to do manually

---

## 1) Users, Groups, and Permissions

Typical structure:
- Groups: `admins`, `developers`, `security-reviewers`
- Project permissions assigned to groups (not individual users)

Best practices:
- Avoid giving admin rights broadly
- Prefer group-based access

---

## 2) Tokens

Token hygiene:
- One token per CI pipeline (or per repo)
- Store tokens in CI secrets/credentials
- Rotate periodically
- Revoke tokens immediately if leaked

---

## 3) Project Provisioning

At scale, aim for:
- consistent naming
- consistent gates/profiles
- consistent onboarding docs

If your org supports it, create templates:
- default `sonar-project.properties` examples
- CI snippets for each platform

---

## 4) Basic Hardening Checklist

- Disable/limit anonymous access (unless intentionally public)
- Use HTTPS (usually via reverse proxy)
- Restrict admin privileges
- Keep SonarQube version updated (prefer LTS)

---

## ✅ Hands-on Lab

1. Create groups for your org.
2. Assign permissions to a project using groups.
3. Create a CI token and store it in a secret manager.
4. Verify that a non-admin user cannot change gates/profiles.

---

## Quick Check

1. Why are group-based permissions better than per-user permissions?
2. Where should tokens live?
3. What are 2 safe-default hardening steps?

---

## Next

Go to **Lesson 21**: [backups and upgrades](21-backups-upgrades-maintenance.md).
