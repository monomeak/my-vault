---
tags: [spring-boot, jpa, flyway, backend, acme-invoice, learning-notes]
date: 2026-07-24
---

# Why a `.sql` file instead of Java entities?

This project's database tables were created with a plain SQL file (`V1__init_schema.sql`) — not by writing Java classes and letting Hibernate figure out the tables. Here's why, in plain terms.

## Two ways to get your tables

**Way 1 — let Hibernate guess.** Write Java classes with `@Entity` on them, tell Spring `ddl-auto=update`, and at startup Hibernate looks at your classes and creates/changes tables to match.

**Way 2 — write the SQL yourself (what this project does).** You write the exact `CREATE TABLE` statements. Hibernate is told `ddl-auto=validate` — it never creates anything, it just double-checks your Java classes match what's already there, and refuses to start if they don't.

## Why Way 2 was picked here

**1. Some things are easier to just write directly.**
Money columns need exact precision (`NUMERIC(19,4)`), some tables need a combined two-column primary key, some columns need specific indexes for speed, and we wanted some starter data (default roles) inserted right away. All of that is one clean SQL file. Getting Hibernate to produce the exact same thing means learning a pile of extra annotations first.

**2. It's safer.**
If you rename a column in a Java class while using Hibernate auto-update, Hibernate can't tell "rename" from "delete this column, add a new one" — and might quietly drop data. A SQL file is instead something a human wrote and can read before it ever runs. Nothing happens that wasn't intentional.

**3. It's the same everywhere.**
A SQL migration file runs identically on your laptop, in a teammate's setup, and eventually in production. Hibernate auto-update instead re-derives the schema from whatever code is currently checked out — which can quietly differ between machines.

**4. Schema first was just simpler for this stage.**
We wanted the database tables ready before designing the actual Java classes for each feature. A SQL file lets that happen independently — the tables exist and are correct, and the Java classes get built later, one feature at a time.

## What happens once Java classes (entities) do get written

Nothing changes about the SQL file. A future `Customer` class just maps onto the `customers` table that's already there:

```java
@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

Because of `ddl-auto=validate`, Spring checks at startup that this class actually matches the real `customers` table. If someone adds a field without updating the database first, the app won't even start — you get a clear error immediately, instead of a confusing bug three weeks later.

## Quick comparison

| | Let Hibernate guess | Write the SQL yourself |
|---|---|---|
| Who's really in charge of the tables | Whatever your Java classes say right now | The `.sql` files, forever |
| Easy to get exact column types/rules | Not really | Yes |
| Starter/seed data | Not supported | Just more SQL |
| Same everywhere (dev, CI, prod) | Not guaranteed | Guaranteed |
| Risk of accidental data loss | Higher | Lower — you review every change |
| Good for | Quick throwaway demos | Anything meant for production |
