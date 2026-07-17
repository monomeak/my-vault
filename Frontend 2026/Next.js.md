---
tags: [nextjs, app-router, framework]
aliases: [Next.js App Router, App Router]
created: 2026-07-17
---

# Next.js

> [!abstract] What it is
> A React framework that adds file-based routing, server rendering, and build-time tooling. This project uses the **App Router** (the `app/` directory), TypeScript, and Tailwind + [[shadcn/ui]].

Related: [[Next.js middleware]] · [[Feature-based architecture]] · [[Auth feature]] · [[React Query]]

---

## File-based routing, quick reference

| File in `app/` | What it does |
|---|---|
| `page.tsx` | The UI for a route — `app/dashboard/page.tsx` → `/dashboard` |
| `layout.tsx` | Shared shell wrapping a route and its children (nav, sidebar) |
| `route.ts` | An API endpoint (Route Handler) instead of a page |
| `loading.tsx` | Shown automatically while `page.tsx` is loading |
| `error.tsx` | Catches errors thrown in that route segment |

```
src/app/
├── layout.tsx          # root layout — wraps everything, mounts AppProviders
├── providers.tsx        # QueryClientProvider + devtools, see React Query
├── admin/
│   └── page.tsx         # example role-gated page (see Next.js middleware)
└── unauthorized/
    └── page.tsx
```

---

## Server Components vs Client Components

> [!question] The default is server-rendered
> Every component in `app/` is a **Server Component** unless it starts with `"use client"`. Server Components run only on the server — no hooks, no browser APIs, no event handlers — and their JS never ships to the browser.

Everything in [[Auth feature]] and [[Profile feature]] that uses `useState`, `useQuery`, or an `onClick` needs `"use client"` at the top:

```tsx
// src/features/auth/components/login-form.tsx
"use client";

import { useState } from "react";
import { useLogin } from "../hooks/use-login";
```

> [!tip] Keep the client boundary as low as possible
> A page can be a Server Component that renders a small Client Component inside it (like `<LoginForm />`) — you don't need to mark the whole page `"use client"` just because one button needs interactivity.

---

## Environment variables

> [!warning] Build-time string replacement, not a runtime lookup
> `NEXT_PUBLIC_*` variables are literally find-and-replaced with their string value at build time. `process.env.NEXT_PUBLIC_API_BASE_URL` works; `process.env[dynamicKey]` does not — the bundler can't statically see it. This project centralizes the read in one place:

```typescript
// src/config/env.ts
function getEnvVar(key: string): string {
  const value = process.env[key];
  if (!value) throw new Error(`Missing required environment variable: ${key}`);
  return value;
}

export const env = {
  apiBaseUrl: getEnvVar("NEXT_PUBLIC_API_BASE_URL"),
} as const;
```

| | Server-only vars | `NEXT_PUBLIC_*` vars |
|---|---|---|
| Readable in | Server Components, Route Handlers | Anywhere, including the browser bundle |
| Use for | secrets, DB URLs | public API URLs, feature flags |
| Risk if misused | — | never put secrets behind this prefix |

---

## Middleware runs before all of this

[[Next.js middleware]] executes before a matched route even reaches `page.tsx` — it's the earliest point in the request lifecycle where you can redirect, rewrite, or block. See that note for how it reads auth cookies set by [[Auth feature]].

---

## See also
- [[Next.js middleware]] — route protection, runs before every page
- [[Feature-based architecture]] — how `src/features/` sits alongside `src/app/`
- [[React Query]] — `AppProviders` client component wrapping the whole app
