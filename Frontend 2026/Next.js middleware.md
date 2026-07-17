---
tags: [nextjs, middleware, auth]
aliases: [Route protection, Middleware]
created: 2026-07-17
---

# Next.js middleware

> [!abstract] What it does
> Runs on the server/edge before a matched route renders. Used here to redirect logged-out users away from protected pages, and gate role-restricted pages like `/admin`.

Related: [[Auth feature]] · [[Feature-based architecture]]

---

## Why it needs cookies, not localStorage

> [!warning] Middleware can't read localStorage
> Middleware runs server-side/edge — it has no access to the browser's `localStorage`. That's why [[Auth feature]] writes an `access_token` + `user_role` cookie at login (`auth-cookies.ts`), purely so middleware has something to check.

```typescript
// src/middleware.ts
export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const token = request.cookies.get("access_token")?.value;
  const role = request.cookies.get("user_role")?.value;

  if (!token && !isPublicPath(pathname)) {
    const loginUrl = new URL("/login", request.url);
    loginUrl.searchParams.set("redirect", pathname);
    return NextResponse.redirect(loginUrl);
  }

  if (token && isPublicPath(pathname)) {
    return NextResponse.redirect(new URL("/", request.url));
  }

  const gate = ROLE_GATED_PATHS.find((g) => pathname.startsWith(g.prefix));
  if (gate && (!role || !gate.allowed.includes(role))) {
    return NextResponse.redirect(new URL("/unauthorized", request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/((?!_next/static|_next/image|favicon.ico|api).*)"],
};
```

---

## What it is *not*

> [!question] A UX gate, not the only security boundary
> Middleware can't validate the token against the real backend on every request — too expensive for edge middleware. It just keeps logged-out users out of the app shell quickly. Any data-mutating request still needs the API itself to check the token server-side.

---

## Production upgrade path

The cookies are currently set from client JS (`document.cookie`), same XSS exposure as localStorage. The real fix: a Next.js Route Handler that proxies login server-side and sets an `httpOnly` cookie via `Set-Cookie` — so client JS (and any injected script) never touches the raw token.

---

## See also
- [[Auth feature]] — where the cookies are written and cleared
