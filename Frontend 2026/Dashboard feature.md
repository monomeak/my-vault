---
tags: [nextjs, react-query, feature-architecture, dashboard]
aliases: [Overview dashboard, Invoice dashboard]
created: 2026-07-17
---

# Dashboard feature

> [!abstract] What it does
> The Overview page for the Acme Invoice Management System — summary stats, revenue chart, invoice status breakdown, and a latest-invoices table.

Related: [[React Query]] · [[Feature-based architecture]]

---

## Folder structure

```
src/features/dashboard/
├── api/
│   └── dashboard-api.ts            # currently mock-only — see note below
├── hooks/
│   └── use-dashboard-overview.ts   # useQuery, seeded with initialData
├── lib/
│   ├── query-keys.ts               # dashboardKeys.all / .overview()
│   ├── format.ts                   # formatCurrency / formatDate
│   └── invoice-status-style.ts     # single source of truth: status -> color/label
├── data/
│   └── dashboard-overview-data.ts  # mock DashboardOverviewData
├── types/
│   └── dashboard.ts                # DashboardStat, RevenuePoint, Invoice, etc.
├── components/
│   ├── dashboard-stat-card.tsx
│   ├── revenue-overview-chart.tsx  # recharts area chart, paid vs pending
│   ├── invoice-status-card.tsx     # stacked bar + legend
│   ├── latest-invoices.tsx         # shadcn Table
│   └── invoice-status-badge.tsx    # shared between status card + table
└── views/
    └── dashboard-overview-view.tsx # composes all three sections
```

---

## Being honest about the mock data

> [!warning] There is no real `/dashboard/overview` endpoint
> DummyJSON has no invoicing data — it only has products, users, carts, posts, etc. Earlier drafts of `dashboard-api.ts` faked a `fetch("/dashboard/overview")` call that would 404 every time. That was corrected:

```typescript
// src/features/dashboard/api/dashboard-api.ts
export async function fetchDashboardOverview(): Promise<DashboardOverviewData> {
  // Simulated network delay so loading states are visible/testable in the UI.
  await new Promise((resolve) => setTimeout(resolve, 300));
  return dashboardOverviewData;
}
```

When a real backend exists, only this function's body changes — the hook, view, and every component stay untouched, because they only ever depended on the `DashboardOverviewData` shape.

---

## `initialData` avoids the loading flash

```typescript
// src/features/dashboard/hooks/use-dashboard-overview.ts
export function useDashboardOverview() {
  return useQuery<DashboardOverviewData>({
    queryKey: dashboardKeys.overview(),
    queryFn: fetchDashboardOverview,
    initialData: dashboardOverviewData,
    staleTime: 60 * 1000,
  });
}
```

See [[React Query]] → "initialData — skipping the loading flash" for why this matters.

---

## Layout spec

```tsx
<div className="space-y-6">
  <section className="grid gap-4 sm:grid-cols-2 xl:grid-cols-4">
    {/* four DashboardStatCard */}
  </section>
  <section className="grid gap-6 xl:grid-cols-[minmax(0,2fr)_minmax(320px,1fr)]">
    {/* RevenueOverviewChart, InvoiceStatusCard */}
  </section>
  <section>
    {/* LatestInvoices */}
  </section>
</div>
```

> [!tip] One new dependency
> `revenue-overview-chart.tsx` uses `recharts` — a charting library, not a UI kit, so it doesn't conflict with "don't introduce another UI library." Install with `pnpm add recharts`.

---

## See also
- [[React Query]] — query pattern, `initialData`, mock-to-real swap strategy
- [[Feature-based architecture]] — why `data/` sits alongside `api/` here (temporary, until a real endpoint exists)
