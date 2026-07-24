---
tags: [flyway, database, migrations, backend, acme-invoice, learning-notes]
date: 2026-07-24
---

# Flyway, explained simply

Flyway's whole job: keep a folder of `.sql` files, remember which ones have already run, and run the new ones — in order, once each, forever.

## The file name actually means something

```
V1__init_schema.sql
│ │  └─ description (your choice, just for humans)
│ └─ two underscores, always — this is required
└─ version number
```

- `V` means "versioned migration" — a one-time change.
- The number decides the order it runs in.
- Two underscores are mandatory — one underscore or a dash and Flyway won't recognize the file.

There are two other, less common prefixes:
- `R` (repeatable) — re-runs every time its content changes, useful for things you want to fully redefine rather than incrementally patch.
- `U` (undo) — a manual "reverse" for a specific migration. Not used in this project.

## How Flyway remembers what it's already done

The first time it runs, Flyway creates its own tracking table, `flyway_schema_history`, and adds one row per migration it successfully runs — including a stored fingerprint (checksum) of that file's contents at the time.

This is why running `./mvnw spring-boot:run` a second time doesn't re-run `V1` again — Flyway checks this table first and sees it's already done.

## Why you can't edit a migration after it's run

If you go back and change `V1__init_schema.sql` after it's already been applied somewhere, Flyway notices the fingerprint doesn't match anymore and refuses to run anything else, until it's fixed. This is on purpose: once a migration has run anywhere (your machine, a teammate's, production), it's considered permanent history. Need to change something? Add a *new* file (`V2__...`) instead of editing an old one.

## `baseline-on-migrate` — what it's for

This project has it turned on, but it only matters in one specific case: pointing Flyway at a database that *already has tables in it*, created some other way before Flyway ever got involved. It tells Flyway "don't panic, just treat everything that's already there as already done, and start managing new changes from here." For a brand-new database (this project's normal case so far), it has no visible effect.

## Commands you'll actually use

```bash
./mvnw flyway:info       # shows every migration: done? pending? when?
./mvnw flyway:validate   # checks nothing's been edited after the fact
./mvnw flyway:repair     # fixes checksum mismatches
./mvnw flyway:clean      # wipes EVERYTHING — dev use only, never production
```

Day to day you won't even need these — Spring Boot runs `migrate` automatically every time the app starts.

## Adding a change later

Say you need a new column on `users`. You never touch `V1`. You add a new file:

```sql
-- V2__add_email_verified_to_users.sql
ALTER TABLE users ADD COLUMN email_verified BOOLEAN NOT NULL DEFAULT false;
```

Restart the app, and Flyway picks it up automatically. Now there are two rows in the history table, and the whole story of how the schema evolved is just the list of files in `db/migration/`, in order.
