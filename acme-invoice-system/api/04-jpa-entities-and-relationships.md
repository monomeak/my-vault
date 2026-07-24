---
tags: [spring-boot, jpa, hibernate, backend, acme-invoice, learning-notes]
date: 2026-07-24
---

# JPA entities and relationships, simply

An "entity" is just a Java class that represents one row in a database table. This note shows what those classes will look like for this project, and how relationships between tables (like "an invoice belongs to a customer") get written in Java.

None of this code exists yet — it's the next step, now that the database tables are ready.

## The basics

```java
@Entity
@Table(name = "customers")
public class Customer {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;
}
```

- `@Entity` — "this class is linked to a database table."
- `@Table(name = "customers")` — which table exactly.
- `@Id` — this field is the primary key.
- `@GeneratedValue(strategy = GenerationType.IDENTITY)` — let the database pick the ID number automatically (matches how the `customers` table was already set up).

That's genuinely most of what you need for a simple entity with no relationships.

## Relationships — how tables connect in Java

This project's schema already has 5 relationships. Here's each one, plain and simple.

### "Many invoices belong to one customer"

```java
// inside Invoice.java
@ManyToOne
@JoinColumn(name = "customer_id")
private Customer customer;
```

`@ManyToOne` means "many of these (invoices) point to one of those (a customer)." `@JoinColumn` says which column holds that link — matches the `customer_id` column that's already in the `invoices` table.

Same pattern, three more times:

```java
// InvoiceItem.java — an item belongs to one invoice, and one product
@ManyToOne @JoinColumn(name = "invoice_id") private Invoice invoice;
@ManyToOne @JoinColumn(name = "product_id") private Product product;

// Payment.java — a payment belongs to one invoice
@ManyToOne @JoinColumn(name = "invoice_id") private Invoice invoice;
```

### "Users can have many roles, and roles can belong to many users"

This one's different — it's a **many-to-many**, so it needs a middle table (`user_roles`) to connect them both ways:

```java
@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles;
}
```

`@JoinTable` says "here's the middle table, and here are its two columns." This has to match the `user_roles` table exactly, or the app won't start (thanks to `ddl-auto=validate`, explained in [[03-why-flyway-sql-instead-of-jpa-entities]]).

## What if Hibernate had created these tables instead?

It would do a decent job automatically — a column for each `@ManyToOne`, a foreign key constraint, a middle table for the `@ManyToMany`. What it wouldn't do on its own: pick the right precision for money columns, add the extra performance indexes, or insert the starter role data. You *can* get Hibernate to do all of that too, just with a lot more annotations — at which point writing the plain SQL directly (what this project does) ends up simpler.

## Why this setup catches mistakes early

Because `ddl-auto=validate` is on, every time the app starts, it double-checks: does every entity class actually match the real database? If someone adds a field to `Customer` and forgets to add the matching column to the database, the app refuses to start — with a clear error message — instead of failing weirdly later when that field actually gets used. Database is the source of truth; Java classes just describe it.
