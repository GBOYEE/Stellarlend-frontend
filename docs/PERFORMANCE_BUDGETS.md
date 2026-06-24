# Performance Budget: Lending Route

To maintain fast initial load times for the lending page, we have implemented code splitting using `next/dynamic`.

## Budget

The lending route chunk is budgeted at **150kb**. 

- This ensures that only the essential code for the landing state (Lending tab) is loaded initially.
- The `BorrowingForm`, `InterestCalculator`, and `ConfirmModal` are loaded lazily on demand.

## How to maintain this budget

If you add new functionality to the lending page:

1. **Lazy Load:** If the new component is not needed for the initial render (e.g., modals, form variants for off-screen tabs), lazy-load it using `next/dynamic`.
2. **Review Imports:** Ensure large libraries or heavy utility functions are not bundled into the main `page.tsx` chunk unnecessarily.
3. **Check CI:** The `bundlewatch` CI check will automatically fail if the budget is exceeded.
4. **Update Budget:** If the budget is legitimately exceeded due to unavoidable new feature requirements, update the `maxSize` in `bundlewatch.config.json` and document the reason for the increase in this file.

---

# Performance Budget: Dashboard Route

The dashboard is the most data-heavy authenticated page. As charts and live data are added, it is most likely to regress. The following Lighthouse CI assertion budgets are enforced on every PR.

## Dashboard Assertion Budgets

| Metric | Threshold | Level | Rationale |
|--------|-----------|-------|-----------|
| LCP (Largest Contentful Paint) | ≤ 3000ms | warn | Dashboard has chart widgets; 3s ceiling catches severe regressions while the page hydrates |
| CLS (Cumulative Layout Shift) | ≤ 0.1 | warn | Live data loading can cause layout shifts; threshold prevents user-visible jank |
| TBT (Total Blocking Time) | ≤ 300ms | warn | Charts and data-fetching can produce long tasks; 300ms keeps interaction responsive |
| Performance Score | ≥ 0.8 | warn | General performance floor to catch regressions from heavy charting libraries |
| Accessibility Score | ≥ 0.9 | error | Accessibility is a hard gate — violations block merge |

## Configuration

Budgets are enforced via `lighthouserc.json` (global assertions) and the `performance.yml` workflow which runs against the staging deployment. The dashboard URL (`/dashboard`) is explicitly included in the Lighthouse CI run.

```
URLs tested:
- https://stellarlend-staging.vercel.app
- https://stellarlend-staging.vercel.app/lending
- https://stellarlend-staging.vercel.app/dashboard  ← dashboard budget enforced here
```

## How to update budgets

If a new feature legitimately changes the dashboard's performance profile:

1. Run `npx lhci autorun` locally against a built app to measure the new baseline.
2. Update thresholds in `.github/workflows/performance.yml` comment block and the table above.
3. Record the before/after numbers in the commit message and PR description.
