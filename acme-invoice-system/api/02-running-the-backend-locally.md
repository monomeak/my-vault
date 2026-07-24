---
tags: [spring-boot, backend, acme-invoice, learning-notes]
date: 2026-07-24
---

# Running `apps/api` locally (no Docker)

## What you need installed

| Tool | Why | Check |
|---|---|---|
| **Java 21** | The app is written for it | `java -version` |
| **PostgreSQL** (running somewhere reachable) | The app needs a real database, no fake fallback | `pg_isready -h localhost -p 5432` |
| **Internet access** (first time only) | To download Maven + all dependencies | — |

You do **not** need Maven installed separately (the project brings its own, see [[00-spring-boot-from-scratch-beginner-guide]]), and you do **not** need Docker — that's just an alternative way to get a database.

## One-time setup: create the database

Postgres needs a database and a login role to exist before the app can use them:

```bash
psql -U postgres -c "CREATE ROLE acme WITH LOGIN PASSWORD 'change_me';"
psql -U postgres -c "CREATE DATABASE acme_invoice OWNER acme;"
```

Match these to whatever's in `application.properties` — that's where the app looks up the database name, username, and password.

## Run it

```bash
cd apps/api
./mvnw spring-boot:run
```

One command. No separate build step.

## Reading the log

A successful start looks roughly like this:

```
Database info:
        Database JDBC URL [jdbc:postgresql://localhost:5432/acme_dev]
Using generated security password: a65e6a3a-53e8-...
Tomcat started on port 8080
Started ApiApplication in 2.928 seconds
```

- **`Database info`** — confirms which database it actually connected to. Good sanity check.
- **`Using generated security password`** — normal for now. There's no real login system built yet, so Spring Security makes up one temporary password each time you restart, just so *something* is protected. Ignore it until the real `auth` feature is built.
- **`Started ApiApplication in X seconds`** — this is the line that means "it's ready." Anything after this, the app is live.

## Check it from outside

```bash
curl http://localhost:8080/actuator/health
# {"status":"UP"}

curl -i http://localhost:8080/anything-else
# 403 — proves the security rules are actually working
```

## Stop it

`Ctrl+C` in the terminal, or `pkill -f "spring-boot:run"` if it's running in the background.

## Common error: password authentication failed

```
FATAL: password authentication failed for user "posgres"
```

This means the username or password in `application.properties` doesn't match what Postgres actually has (or there's a typo, like `posgres` instead of `postgres`). Fix it there, rerun — no rebuild needed.

## Prefer Docker instead?

```bash
docker compose up -d postgres     # just the database, containerized
cd apps/api && ./mvnw spring-boot:run   # app still runs normally, talks to that container
```
