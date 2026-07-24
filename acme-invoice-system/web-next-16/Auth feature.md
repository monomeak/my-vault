---
tags: [nextjs, auth, react-query, feature-architecture]
aliases: [Authentication, Login flow]
created: 2026-07-17
---

# Auth feature

> [!abstract] What it does
> Login, register, and forgot-password flows for the app, built against DummyJSON, with session persistence via localStorage + cookies (so [[Next.js middleware]] can gate routes).

Related: [[React Query]] · [[Next.js middleware]] · [[Feature-based architecture]] · [[Profile feature]] · [[Zod]]

---

## Folder structure

```
src/features/auth/
├── api/
│   └── auth-api.ts            # login, register, forgotPassword, resetPassword, refreshToken, fetchCurrentUser
├── hooks/
│   ├── use-login.ts           # useMutation — validates, calls API, fetches role, persists session
│   ├── use-register.ts        # useMutation — validates, per-field errors
│   ├── use-forgot-password.ts # useMutation — simulated request (no real endpoint yet)
│   ├── use-current-user.ts    # useQuery — cached current user, seeded on login
│   └── use-auth-session.ts    # logout() — clears storage, cookies, and query cache
├── lib/
│   ├── session-storage.ts     # localStorage adapter for the session
│   ├── auth-cookies.ts        # sets/clears cookies so middleware can read auth state
│   └── query-keys.ts          # authKeys.all / .currentUser()
├── mappers/
│   ├── auth.mapper.ts         # DTO -> AuthSession / AuthUser
│   └── role.mapper.ts         # ApiRole <-> AppRole mapping + hasMinimumRole()
├── schemas/
│   ├── login.schema.ts
│   ├── register.schema.ts
│   └── forgot-password.schema.ts
├── types/
│   ├── dummy-auth.ts           # raw DummyJSON response shapes
│   └── auth.ts                 # AuthUser, AuthSession, payload types, ApiRole/AppRole
└── components/
    ├── login-form.tsx
    ├── register-form.tsx
    └── forgot-password-form.tsx
```

---

## The role mapping problem

DummyJSON returns roles as `admin | user | moderator`. The app's domain wants `super_admin | shop_admin | staff`. This is centralized in one file so there's a single source of truth:

```typescript
// src/features/auth/mappers/role.mapper.ts
const ROLE_MAP: Record<ApiRole, AppRole> = {
  admin: "super_admin",
  moderator: "shop_admin",
  user: "staff",
};
```

> [!tip] Why `Record<ApiRole, AppRole>`
> TypeScript forces every case to be handled — forget a role in the map and it's a compile error, not a runtime surprise.

`hasMinimumRole(userRole, required)` builds a simple permission check on top, using a hierarchy (`super_admin` > `shop_admin` > `staff`).

---

## Login flow, end to end

1. `login-form.tsx` calls `useLogin()`'s `mutateAsync`
2. `use-login.ts` validates with `loginRequestSchema` ([[Zod]])
3. Calls `loginRequest()` in `auth-api.ts` → DummyJSON `/auth/login`
4. Since the login response has no `role`, a second call hits `/auth/me` to get it
5. `mapToAuthSession()` builds the domain `AuthSession`
6. Session is saved to **both**:
   - `localStorage` (via `session-storage.ts`) — for client-side reads
   - **cookies** (via `auth-cookies.ts`) — so [[Next.js middleware]] can check auth server-side
7. `onSuccess` seeds the `["auth","me"]` [[React Query]] cache instantly — no extra fetch needed

> [!warning] Cookie security caveat
> Cookies here are set from client JS (`document.cookie`), which means they're readable by any XSS payload — same exposure as localStorage. For real production auth, the backend should set `httpOnly` cookies via a `Set-Cookie` header (e.g. through a Next.js Route Handler acting as a BFF), so client JS never touches the raw token.

---

## Register and forgot-password: honesty about mock limits

> [!question] DummyJSON doesn't actually support these
> `/users/add` doesn't persist a real account — it echoes back a fake object. There's no real forgot-password endpoint at all. Both `registerRequest()` and `forgotPasswordRequest()` are written so the **hook and component contracts are final**, but the underlying call is a stand-in until a real backend exists.

```typescript
// src/features/auth/api/auth-api.ts
export async function forgotPasswordRequest(payload: ForgotPasswordPayload) {
  // No real DummyJSON endpoint — simulated so the UI/hook layer is ready
  await new Promise((resolve) => setTimeout(resolve, 800));
  return { message: `If an account exists for ${payload.email}, a reset link has been sent.` };
}
```

---

## Auto-login after register?

> [!example] Design decision, not yet wired
> DummyJSON's register response has no tokens, so "auto-login after register" would mean calling `useLogin()` right after `useRegister()` succeeds, using the same credentials. Left as a follow-up — see [[React Query]] for the `mutateAsync` chaining pattern this would use.

---

## See also
- [[React Query]] — the `useMutation`/`useQuery` patterns used throughout these hooks
- [[Next.js middleware]] — how `access_token`/`user_role` cookies gate protected routes
- [[Profile feature]] — reuses `role.mapper.ts` from this feature
- [[Zod]] — schema validation for login/register/forgot-password payloads
