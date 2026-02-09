# Workspace AGENTS.md

## Purpose
This is the top-level operator manual for the whole workspace.
Use this file first if you start in `/Users/petergelgor/Documents/projects/budgeting_app` and do not yet know whether a change belongs to frontend, backend, or both.

## Repository Topology (Critical)
- This workspace is a container folder, not a git repository.
- `frontend/` is its own git repository.
- `backend/` is its own git repository.
- Commit and branch operations must be run separately inside each repo.

## Directory Map
- Frontend app: `/Users/petergelgor/Documents/projects/budgeting_app/frontend`
- Backend API: `/Users/petergelgor/Documents/projects/budgeting_app/backend`
- Shared local orchestration: `/Users/petergelgor/Documents/projects/budgeting_app/docker-compose.yml`
- Workspace search defaults: `/Users/petergelgor/Documents/projects/budgeting_app/.rgignore`

## First-Read Order For New Agents
1. Read `/Users/petergelgor/Documents/projects/budgeting_app/AGENTS.md` (this file).
2. Read `/Users/petergelgor/Documents/projects/budgeting_app/frontend/AGENTS.md` for UI/runtime/query architecture.
3. Read `/Users/petergelgor/Documents/projects/budgeting_app/backend/AGENTS.md` for API/security/data architecture.

## Runtime Modes

### Mode A: Docker full stack (recommended for integrated behavior)
- Start from workspace root:
  - `docker compose up --build -d`
- Exposed host ports:
  - Frontend: `http://localhost:3000`
  - Postgres: `localhost:5432`
- Backend is not published directly on host in root compose; browser traffic reaches backend through frontend Nginx proxy (`/api/*`).

### Mode B: Split local dev (frontend Vite + backend Spring)
- Start backend in `/backend` (typically `mvn spring-boot:run`).
- Start frontend in `/frontend` (`npm run dev`).
- Frontend `.env` should point to `VITE_API_BASE_URL=http://localhost:8080/api/v1`.
- CORS must allow `http://localhost:5173` on backend side.

## Cross-Stack Request Path

### In Docker full stack
1. Browser calls `http://localhost:3000/api/v1/...`.
2. Frontend Nginx proxies `/api/*` to backend service on Docker network.
3. Backend handles request and returns JSON.
4. Frontend `HttpClient` validates payload with Zod schemas before exposing data to pages.
5. Backend local profile startup runs categorization-rule tracking backfill for existing transactions.

### In Vite + backend split mode
1. Frontend `HttpClient` calls `VITE_API_BASE_URL` directly.
2. Browser sends Authorization header if access token exists in session store.
3. On `401`, frontend runs one refresh attempt, retries original request once.

## End-to-End Auth Flow (Frontend + Backend)
1. User logs in/registers on frontend auth page.
2. Frontend calls `/api/v1/auth/login` or `/api/v1/auth/register`.
3. Backend returns access token + refresh token + user object.
4. Frontend stores access token in-memory and refresh token in localStorage.
5. Protected routes render only if auth state is `authenticated`.
6. On page reload, frontend calls `/api/v1/auth/refresh` then `/api/v1/auth/me`.
7. On expired/invalid refresh token, frontend clears session and redirects to `/login` for protected pages.

## Frontend Route To Backend API Ownership
- `/login`, `/register`
  - Uses backend auth endpoints only.
- `/dashboard`
  - Uses `/accounts/summary`, `/analytics/cashflow`, `/analytics/spending`, `/categories?flat=true`.
- `/connections`
  - Uses `/connections`, `/connections/simplefin/setup`, `/connections/{id}/sync`, `/connections/{id}` (DELETE), `/accounts/summary`.
- `/transactions`
  - Uses `/transactions` (GET/POST), `/transactions/coverage`, `/transactions/{id}` (PATCH/DELETE), `/accounts`, `/categories?flat=true`, `/categorization-rules` (POST).
- `/categories`
  - Uses `/categories` GET/POST/PUT/DELETE and `/categorization-rules` GET/POST/PUT/DELETE and `/categorization-rules/{id}/transactions` (GET) and `/categorization-rules/backfill` (POST, maintenance).
- `/budgets`
  - Uses `/budgets` GET/PUT/DELETE, `/categories?flat=true`, `/analytics/spending`, `/transactions`.
- `/recurring`
  - Uses `/recurring` GET, `/recurring/detect` POST, `/recurring/upcoming` GET, `/recurring/{id}` PATCH/DELETE.
- `/accounts/:id`
  - Uses `/accounts/{id}` GET, `/accounts/{id}/net-worth-category` PATCH, `/transactions` GET (filtered by accountId).

## Current Product Contract Boundaries
- Backend is source of truth for users, connections, accounts, categories, transactions, transfers, analytics, recurring patterns, and budget targets.
- All backend endpoint areas are now surfaced in frontend UI.

## Non-Negotiable Invariants
- Frontend is TS-only in canonical source/test paths.
  - No generated `.js` in `frontend/src` or `frontend/tests/e2e`.
  - Enforced by `frontend/scripts/check-no-generated-js.mjs` and `npm run check:no-generated-js`.
- Canonical frontend routes live in `frontend/src/app/routes.ts`.
- Canonical frontend query keys live in `frontend/src/shared/query/keys.ts`.
- Canonical month-date helper is `frontend/src/shared/utils/date.ts`.
- Backend controller payloads should be typed DTOs, not generic `Map` request bodies.
- Backend transaction storage access is split by concern:
  - read: `TransactionReadRepository`
  - write: `TransactionWriteRepository`
  - analytics projections: `TransactionAnalyticsRepository`

## Search Recipes (Fastest Reliable Queries)
- Find all AGENTS docs:
  - `rg --files -g '**/AGENTS.md'`
- Frontend TypeScript only:
  - `rg -n "<pattern>" frontend/src -g '*.ts' -g '*.tsx'`
- Backend Java only:
  - `rg -n "<pattern>" backend/src/main/java -g '*.java'`
- Backend endpoint declarations:
  - `rg -n "@RequestMapping|@GetMapping|@PostMapping|@PutMapping|@PatchMapping|@DeleteMapping" backend/src/main/java/com/peter/budget/controller`
- Frontend request cache usage:
  - `rg -n "queryKeys\\.|useQuery|invalidateQueries" frontend/src -g '*.ts' -g '*.tsx'`

## Validation Commands

### Frontend
- `npm run check:no-generated-js`
- `npm run typecheck`
- `npm run test`
- `npm run build`
- `npm run e2e`
- `npx playwright test --list` (quick duplicate-spec sanity check)

### Playwright MCP (Required For New/Changed UI Features)
- After implementing any user-facing frontend feature, run a manual smoke flow with Playwright MCP against the running app and report pass/fail status in the handoff.
- For features that depend on backend data/auth/database state, run MCP against Docker full stack started from workspace root with `docker compose up --build -d` (so Postgres and backend are active).
- If login is required and credentials were not provided in the thread, ask the user for username/password before running the MCP browser flow.
- Do not mark a UI feature as confirmed without this MCP validation status.

### Backend
- `mvn test`
- `mvn -DskipTests package`

## Notes And Learnings Protocol
- Frontend notes location: `/Users/petergelgor/Documents/projects/budgeting_app/frontend/notes_to_agent`
- Backend notes location: `/Users/petergelgor/Documents/projects/budgeting_app/backend/notes_to_agent`
- If you add a note in either repo, update that repo’s `notes_to_agent/index.md` in the same change.

## Known Functional Gaps (Important)
- No standalone categorization-rule management page (rules managed inline from Transactions and Categories pages).
