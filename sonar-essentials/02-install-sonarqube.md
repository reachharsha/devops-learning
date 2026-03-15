---
render_with_liquid: false
---
# 02 — Install SonarQube (Docker) + SonarCloud Setup

This lesson gets you a working Sonar environment.

- If you want **self-hosted**: install **SonarQube** locally using Docker.
- If you want **hosted**: set up **SonarCloud** and connect your Git provider.

---

## 🎯 Learning Objectives

- Run SonarQube locally using Docker Compose
- Understand the minimum requirements (DB, memory)
- Log in and complete initial setup safely
- Create a SonarCloud org/project (optional path)

---

## Option A — SonarQube with Docker Compose (Recommended for Learning)

### Why Compose?
- Reproducible setup
- Easy reset
- Mirrors real deployments (server + DB)

### 1) Create a local Compose file

Create a folder like `sonarqube-local/` and add `docker-compose.yml`:

```yaml
services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: sonar
      POSTGRES_PASSWORD: sonar
      POSTGRES_DB: sonarqube
    volumes:
      - sonar_db:/var/lib/postgresql/data

  sonarqube:
    image: sonarqube:lts-community
    depends_on:
      - db
    ports:
      - "9000:9000"
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/sonarqube
      SONAR_JDBC_USERNAME: sonar
      SONAR_JDBC_PASSWORD: sonar
    volumes:
      - sonar_data:/opt/sonarqube/data
      - sonar_extensions:/opt/sonarqube/extensions
      - sonar_logs:/opt/sonarqube/logs

volumes:
  sonar_db:
  sonar_data:
  sonar_extensions:
  sonar_logs:
```

Start it:

```bash
docker compose up -d
```

Open:
- http://localhost:9000

### 2) First login
Default credentials:
- Username: `admin`
- Password: `admin`

You’ll be prompted to change the admin password.

### 3) Health checks
Useful commands:

```bash
docker compose ps
docker compose logs -f sonarqube
```

If SonarQube is slow to start, wait—initial boot can take a bit.

---

## Common Setup Issues (and fixes)

### 1) Not enough memory
SonarQube needs a decent amount of RAM (especially with Docker Desktop). If the UI doesn’t load:
- Increase Docker Desktop memory allocation (e.g., 4–6GB)

### 2) Elasticsearch bootstrap checks
SonarQube uses Elasticsearch internally.

On Linux servers, you may need:
- `vm.max_map_count=262144`

For macOS/Windows Docker Desktop, this is usually handled by the VM.

### 3) DB connectivity problems
If you changed passwords/DB name, update the JDBC env vars and restart:

```bash
docker compose down
docker compose up -d
```

---

## Option B — SonarCloud (Hosted)

SonarCloud is ideal when:
- You don’t want to run servers
- Your code is on GitHub/GitLab/Bitbucket

Typical setup:
1. Log into SonarCloud
2. Create an Organization
3. Import a repository
4. Configure CI to run analysis

You’ll generate a token and store it in your CI secrets.

---

## ✅ Hands-on Lab

1. Bring up SonarQube with Compose.
2. Log in and change the admin password.
3. Click around the UI and locate:
   - Projects
   - Quality Profiles
   - Quality Gates
   - Administration

---

## Quick Check

1. Why is Postgres commonly used with SonarQube?
2. What port does SonarQube UI use by default?
3. Where should CI tokens live (in code or secrets)?

---

## Next

Go to **Lesson 03**: Your first scan.
