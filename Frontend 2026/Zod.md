---
tags: [validation, typescript]
aliases: [Zod schemas]
created: 2026-07-17
---

# Zod

> [!abstract] What it does
> Runtime schema validation for anything crossing a trust boundary: API responses, form input, environment variables. TypeScript types disappear at runtime — Zod is what actually checks the shape when the data arrives.

Related: [[Auth feature]] · [[Profile feature]]

---

## Pattern used across this project

Every feature validates at the network boundary, right where the raw response is parsed:

```typescript
// src/features/profile/api/profile-api.ts
const json = await res.json();
const parsed = profileSchema.safeParse(json);
if (!parsed.success) {
  console.error(parsed.error.flatten());
  throw new Error("Invalid profile response shape");
}
return parsed.data;
```

> [!tip] `safeParse` over `parse`
> `safeParse` never throws — it returns `{ success, data }` or `{ success: false, error }`, so you control the failure path instead of an uncaught exception crashing the fetch.

---

## Also used for form/payload validation

```typescript
// src/features/auth/schemas/register.schema.ts
export const registerRequestSchema = z
  .object({
    username: z.string().min(3),
    email: z.string().email(),
    password: z.string().min(6),
    confirmPassword: z.string(),
  })
  .refine((data) => data.password === data.confirmPassword, {
    message: "Passwords do not match",
    path: ["confirmPassword"],
  });
```

`.refine()` adds cross-field validation Zod's base object schema can't express alone — here, comparing two fields against each other.

---

## `z.infer` — one schema, one type

```typescript
export type RegisterRequest = z.infer<typeof registerRequestSchema>;
```

The TypeScript type is *derived from* the schema, not hand-duplicated next to it — so the validation rules and the type can never drift out of sync.

---

## See also
- [[Auth feature]] — login/register/forgot-password schemas
- [[Profile feature]] — validates the DummyJSON user response
