**ACME Invoice Management System** — a learning-focused, production-shaped monorepo for managing invoices, customers, payments, teams, analytics, and reports.

**Structure**: pnpm workspace + Turborepo, with one active app: `apps/web`.

**Stage 1 (current)** — frontend prototype:

- Next.js 16 (App Router), React 19, TypeScript 5
- Tailwind CSS 4 + shadcn/ui + Base UI for styling
- TanStack React Query 5 for server state, Zod 4 for validation
- Recharts for dashboard charts, Sonner for notifications
- Talks to **DummyJSON** as a temporary fake backend (auth, users)
- Feature-based architecture under `apps/web/src/features/` (auth, dashboard-overview, invoices, profile), each following: route → view → query hook → API function → Zod validation/mapper → DummyJSON

**Stage 2 (planned, not started)** — real backend:

- Java 21 + Spring Boot + PostgreSQL + Flyway, exposed via OpenAPI, incrementally replacing DummyJSON feature by feature