# Budgeting App

This repo contains:

- `backend`: Spring Boot API
- `frontend`: React app
- `postgres`: local persistent database (Docker volume)

## Current Product Status (Audited Feb 7, 2026)

Legend:
- `[WORKING]`: implemented and verified in this audit
- `[PARTIAL]`: implemented, but depends on external systems or has known limits
- `[MISSING]`: not implemented or not exposed for users yet

Verification run during this audit:

```bash
# Backend
cd backend
mvn test
mvn -DskipTests package

# Frontend
cd frontend
npm test
npm run typecheck
```

Commands above passed. Docker image builds for both `backend` and `frontend` also passed.

## Feature Inventory

### Frontend (what you can use right now)

| Feature | Status | Notes |
|---|---|---|
| Register / Login / Logout | `[WORKING]` | Full auth flow works and redirects correctly. |
| Session restore on app reload | `[WORKING]` | Refresh-token-based restore is wired and tested. |
| Protected routes | `[WORKING]` | Unauthenticated users are redirected to `/login`. |
| Dashboard (net worth, cashflow, spending) | `[WORKING]` | Uses backend analytics + account summary endpoints. |
| Connections screen (list/add/sync/remove) | `[WORKING]` | Verified with real data; institution label now aggregates correctly when multiple institutions are present. |
| Transactions list, filters, pagination | `[WORKING]` | Verified with real data; account/category/date filters and paging update results correctly. |
| Transactions history coverage card | `[WORKING]` | Shows total imported count plus oldest/newest imported transaction dates. |
| Transaction inline edits (category/notes/exclude) | `[WORKING]` | Category assign/update/clear works (`categoryId: null` clears back to uncategorized). |
| Categories CRUD (custom categories) | `[WORKING]` | Create/update/delete custom categories works; system categories are protected. |
| Budgets page (monthly targets, variance, warnings) | `[WORKING]` | Uses persisted backend budgets (`GET/PUT/DELETE /api/v1/budgets...`) so targets sync across browsers/devices. |
| Server-backed budgets | `[WORKING]` | Implemented end-to-end with DB-backed monthly targets by category. |
| Recurring bills management UI | `[MISSING]` | Backend endpoints exist, but no frontend page/route. |
| Transfer pairing/unlinking UI | `[MISSING]` | Backend endpoints exist, but no frontend controls. |

### Backend API (what exists today)

| API Area | Status | Notes |
|---|---|---|
| Auth: register, login, refresh, logout, me | `[WORKING]` | Verified with live requests including refresh revocation on logout. |
| Categories: list/get/create/update/delete | `[WORKING]` | Verified including 404s and system-category protections. |
| Accounts: list, summary, get by id | `[WORKING]` | Verified list/summary; not represented by dedicated UI pages beyond summary usage. |
| Transactions: list/get/update | `[WORKING]` | Verified list/get/update paths and validation behavior, including category clear via `categoryId: null`. |
| Transactions coverage: `/transactions/coverage` | `[WORKING]` | Returns imported transaction count and min/max posted dates for current user. |
| Transfer APIs: list/mark/unlink | `[PARTIAL]` | Endpoints exist; full pair/unpair flow not fully exercised here due no seeded transfer data. |
| Analytics: spending/trends/cashflow | `[WORKING]` | Verified with empty dataset responses and valid shapes. |
| Recurring: list/detect/upcoming/calendar/toggle/delete | `[WORKING]` | Endpoints available and responding; frontend does not expose them yet. |
| SimpleFIN connection setup/sync | `[PARTIAL]` | Implemented and verified with real data, including historical backfill windows; total history depth still depends on what each institution/provider returns to SimpleFIN. |
| Scheduled sync + recurring jobs | `[PARTIAL]` | Scheduler is enabled and runs, but production behavior depends on real connections/data. |
| Auto-categorization rules engine | `[PARTIAL]` | Matching engine exists in backend service, but there is no public API/UI to manage rules yet. |
| Budget persistence API | `[WORKING]` | `GET /budgets?month=YYYY-MM`, `PUT /budgets/{month}`, `DELETE /budgets/{month}/categories/{categoryId}` backed by DB table `budget_targets`. |

## Current issues
1. Historical depth beyond the imported coverage window can still be limited by upstream provider data availability through SimpleFIN.

## Known Gaps / Non-Working User Flows

1. No user-facing recurring bills feature, despite backend support.
2. No user-facing transfer pairing/unpairing feature, despite backend support.
3. No user-facing categorization-rule management (rules can only be managed internally/direct DB today).
4. Backend automated tests are still sparse in many areas (improved for transaction category clear behavior in this pass, but broader service/controller coverage is still needed).
5. There is no user-facing "backfill diagnostics" UI to explain why historical imports stop at a specific date.

## Verified SimpleFIN History Findings (Feb 7, 2026)

- UI and API both report imported coverage for the tested account as:
  - total imported: `306`
  - oldest imported: `2025-08-24`
  - newest imported: `2026-02-04`
- Direct provider-window checks against the same SimpleFIN access URL returned:
  - `2025-08-01..2026-02-07`: `306` transactions
  - `2025-01-01..2025-07-31`: `0`
  - `2024-01-01..2024-12-31`: `0`
  - `2023-01-01..2023-12-31`: `0`
  - `2022-01-01..2022-12-31`: `0`
  - `2020-01-01..2021-12-31`: `0`
- Based on this audit, the "about 6 months" history limit is coming from upstream returned data for this current connection, not from transaction page filters.

## Recommended Roadmap

### Phase 1: Close correctness gaps (high impact, low-medium effort)

1. Completed Feb 7, 2026: allow clearing a transaction category (`categoryId: null`) in transaction update flow.
2. Standardize auth/security error payloads for unauthenticated requests.
3. Add backend tests for Auth, Categories, Transactions, Analytics services/controllers.

### Phase 2: Ship hidden backend value in UI

1. Add recurring page (list, detect now, upcoming, toggle active, delete).
2. Add transfer management in transactions UI (view transfer pairs, mark/unlink).
3. Add account detail view using existing `/accounts/{id}` API.

### Phase 3: Budget V2 (server-backed)

1. Completed Feb 7, 2026: implement `/api/v1/budgets` persistence API + DB-backed monthly targets.
2. Completed Feb 7, 2026: migrate frontend budgets/dashboard to API-backed targets instead of local storage.
3. Optional follow-up: add one-time migration/import path for any legacy localStorage budget targets.

### Phase 4: SimpleFIN hardening

1. Add integration tests for setup/sync with controlled fixture responses.
2. Improve sync observability (structured sync logs + per-connection metrics + backfill window counters).
3. Add retry/backoff handling and clearer user-facing sync error surfacing.
4. Add a "history diagnostics" panel that explains oldest imported date and whether older windows were empty.
5. Add optional manual backfill controls (extend cutoff, retry older windows) for institutions that expose deeper archives.

### Phase 5: Quality and release readiness

1. Add API contract tests and frontend integration tests for each core page flow.
2. Add CI gates for `mvn test`, `npm test`, `npm run typecheck`, `npm run build`, `npm run e2e`.
3. Add seed/demo mode for local product walkthrough without real bank connectivity.

## Quick Start (Recommended)

1. Copy the env template:

```bash
cp .env.example .env
```

2. Optionally update `JWT_SECRET` and `ENCRYPTION_SECRET` in `.env`.

3. Start everything:

```bash
docker compose up --build -d
```

Open:

- App: [http://localhost:3000](http://localhost:3000)
- API (via frontend proxy): [http://localhost:3000/api/v1](http://localhost:3000/api/v1)

## Networking Notes

- The backend container is intentionally not exposed on a host port by default.
- Browser API requests go through frontend Nginx (`/api/*` -> backend service on Docker network).
- This avoids local `8080` conflicts and keeps trial setup simpler.

## Useful Commands

```bash
# View container status
docker compose ps

# Tail logs
docker compose logs -f

# Stop stack
docker compose down

# Stop and remove DB data (full reset)
docker compose down -v
```

## Notes

- Flyway migrations run automatically when backend starts.
- PostgreSQL data persists in the `postgres_data` Docker volume.
- Frontend API calls are proxied through Nginx to backend (`/api/*`).
