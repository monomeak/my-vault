---
tags: [spring-boot, backend, acme-invoice, learning-notes]
date: 2026-07-24
---

# Stage 2 backend — what's been built so far

Quick recap: the ACME Invoice project started as a Next.js frontend talking to a fake API (DummyJSON). Stage 2 is building the real backend — Spring Boot + PostgreSQL — at `apps/api`. This note covers what exists right now: a working skeleton, no business features yet.

## How it was created

Generated with the real Spring Initializr service (see [[00-spring-boot-from-scratch-beginner-guide]] for the full explanation), not written by hand — so all the versions and file names are exactly what a real Spring Boot project expects.

One gotcha along the way: Spring Boot's version is `4.0.7`, not `4.0.7.RELEASE` (that `.RELEASE` suffix stopped being used a few years ago, but Initializr's dropdown still shows it that way). Had to fix that by hand in `pom.xml`.

Also worth knowing: this project uses **Spring Boot 4**, which is newer than what most tutorials cover (usually 3.x). A couple of names have already changed — the web starter is `spring-boot-starter-webmvc` now, not `-web`. Small thing, but good to know so it doesn't look like a typo.

## The folders

One Java package per topic:

```
com.acme.invoice/
├── auth/        — login, logout (not built yet)
├── user/        — users & roles
├── customer/    — customers
├── product/     — products
├── invoice/     — invoices & invoice items
├── payment/     — payments
├── dashboard/   — summary stats for the frontend
├── report/      — exports/reports
├── audit/       — logs of who changed what
├── security/    — login/permission rules
├── config/      — small setup files (CORS, API docs)
└── common/      — shared bits used everywhere
```

Right now, most of these are just empty folders with a one-line description — placeholders for real code that comes later.

## What's actually working today

- **Security rules** — `/actuator/health` and the API docs pages are open to everyone; literally everything else needs a login. Strict by default, since there's no login system built yet.
- **CORS setup** — allows the frontend (`localhost:3000`) to call this API from the browser.
- **API docs** — auto-generated, viewable at `/v3/api-docs`.
- **Database schema** — all the tables exist (`users`, `customers`, `invoices`, etc.), created by a Flyway migration file. Why a plain `.sql` file instead of Java classes? See [[03-why-flyway-sql-instead-of-jpa-entities]].
- **Docker setup** — a `Dockerfile` for the app, and a `docker-compose.yml` that starts the app + a Postgres database together.

## How it was double-checked

Rather than just "it compiles," this was actually booted up against a real Postgres database and checked:

- The database tables got created correctly
- `/actuator/health` returned `{"status":"UP"}`
- The API docs page loaded
- Hitting a random made-up URL correctly got blocked (403) — proof the security rules are actually active

## What's not built yet

No real features — no way to create a customer, an invoice, log in, anything. No entities (Java classes representing tables — see [[04-jpa-entities-and-relationships]]). No tests beyond the one Spring Boot generates automatically. This is the "empty building with plumbing installed" stage — next comes actually building rooms.
