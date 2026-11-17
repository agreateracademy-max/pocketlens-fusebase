# Decision: split the stack, do not rewrite it

PocketLens is three layers: a React/Vite SPA, a Python/FastAPI Plaid sync service, and
Supabase (Postgres + GoTrue + PostgREST + RLS).

| Layer | Where it runs | Why |
|---|---|---|
| `web/` React SPA | FuseBase App (frontend slot) | Plain static Vite build; FuseBase serves it on its own subdomain and can embed it in a portal. |
| Postgres + auth + RLS | Supabase (hosted) | The app calls PostgREST directly and every RLS policy uses `auth.uid()`. |
| `finance-backend/sync-service/` | Render / Docker host | Python + FastAPI + cron + Plaid SDK. FuseBase app backends are Node.js. |

Nothing in `web/` changes except the four `VITE_*` values, which are build-time and public.
Upstream stays mergeable.

## Rejected alternatives

- **Port `sync-service` to Node.** ~150 KB of Plaid sync, reconcile, transfer, investment and
  digest logic. A separate project with real regression risk.
- **Move off Supabase to FuseBase Postgres.** Every table policy is written against
  `auth.uid()`; the swap forks the schema from upstream permanently.
- **Run the Python service inside a FuseBase app backend.** Not possible today.

## Later migration path, if FuseBase must own the data layer

1. Move `finance-backend/supabase/migrations/` into a FuseBase isolated store and replace
   `auth.uid()` with the FuseBase app token subject.
2. Replace `@supabase/supabase-js` calls in `web/src/data/` with FuseBase dashboard SDK calls.
3. Port `sync-service` to Node.

One layer at a time, with the Supabase version still running as the fallback.
