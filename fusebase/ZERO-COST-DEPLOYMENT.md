# $0-Only Deployment Plan (2026-11-17)

**Hard requirement: no payments, no paid plans, no paid-tier fallbacks.** This document supersedes the Render-first plan. The GitHub repository remains the source of truth; deployment/runtime changes must be made in GitHub and remain reproducible from it.

## Verified current state

- FuseBase frontend is live at https://pocketlens.thefusebase.app/.
- Supabase project `pocketlens` (`fuabnlnvapowqgerqljt`) exists on Free; 48 SQL migrations applied.
- No Plaid credential or transaction data has been configured in production.
- PocketLens frontend calls a separate API for Link, credential management, sync requests, and Plaid webhook callbacks; it cannot work with only a static frontend.

## Candidate A: Cloudflare Workers Free — primary evaluation target

Cloudflare's current Free limits: 100,000 requests/day; 10 ms active CPU for **both HTTP and cron** invocations; 128 MB memory; 50 subrequests/invocation; 5 cron triggers/account; 15-minute wall-clock cron run. Awaiting HTTP I/O does not count as CPU. Python Workers use Pyodide; FastAPI works via Cloudflare's ASGI adapter, but Python `threading`/`multiprocessing` are nonfunctional. Python packages must resolve against Pyodide/PyEmscripten wheels.

### Actual compatibility experiment (local only)

- FastAPI + Cloudflare ASGI Worker launched and returned `/health` successfully in local workerd.
- The complete PocketLens API package tree was copied into a temporary fit-test, with its generated Plaid SDK source vendored (not modified in GitHub yet).
- `pywrangler sync` fails on the upstream `plaid-python` dependency because it publishes no Pyodide wheel. Vendoring its generated ~38MB pure-Python client allowed its imports and an instance of its `PlaidApi` to initialize successfully. `cryptography` and `PyJWT` also imported/operated in the test.
- With the Plaid SDK vendored and `urllib3` source vendored for this test only, the full FastAPI app booted and `/health` passed in the local Cloudflare runtime.
- This is promising but is NOT a deployed/supported production backend yet. Full `/link/*`, `/sync`, `/webhook`, and cron tests have not passed.

### Primary remaining blocker: code relies on threads and sockets

Upstream FastAPI endpoints and SDK operations use synchronous `plaid-python`, synchronous `supabase-py`, `BackgroundTasks`, and a per-thread Supabase client (`threading.local`). The Plaid generated client uses `multiprocessing.pool.ThreadPool`, `multiprocessing.cpu_count`, and urllib3. Cloudflare Python Workers do not have working threads or multiprocessing; blocking synchronous request libraries are not production-safe without compatibility verification. CPython thread-local client pooling cannot be relied on in one isolate serving concurrent async requests.

**Plan:** use GitHub as base and make the minimal Cloudflare-specific port:

1. Preserve upstream logic and tests under `legacy/` or an explicit branch; do not discard existing behavior.
2. Create a Cloudflare Python Worker using FastAPI (or direct Worker handlers) with actual async HTTP via `httpx`/Fetch for Plaid REST and Supabase PostgREST. Reuse data rules/SQL/RLS; do not expose service-role or Plaid secrets to the SPA.
3. Replace thread-local/global clients and ThreadPool with per-request or task-local context; no shared user secrets/state in module globals.
4. Add authenticated endpoints for Plaid Link token creation, token exchange, personal credential CRUD, item unlink/backfill, plus signed Plaid webhook handling. Persist access tokens encrypted at rest and verify Plaid webhook JWT as upstream does.
5. Make the cron handler incremental-only and idempotent: one account/item per short invocation; checkpoint cursors; queue/chunk/retry. A 10ms CPU cap applies per call, so schedule frequent small chunks rather than a monolithic full reconciliation. Keep full-window reconciliation manual or split into bounded pages only after proving CPU fits.
6. Run a Sandbox test sequence against the existing 48 migrations: login/user isolation, credential encryption roundtrip, Link token, public-token exchange, one `transactions/sync` page, duplicate webhook, and rerun idempotency. Measure CPU; do not enable production Plaid or say it is deployed until all pass.
7. Deploy only using Cloudflare Free features. Keep billing alerts/spending protections on; never switch to Workers Paid ($5+).

Since upstream `plaid-python` has no Pyodide wheel, a vendored SDK is an experimental option but may be large/brittle; the better long-term adaptation is using the generated OpenAPI models while replacing the HTTP transport with Web APIs/async fetch or a proven compatible HTTP client. Make updates reproducible and test against Plaid's current API.

## Candidate B: FuseBase hosted backend — conditional, not yet confirmed $0

Our account-specific 20-code document reports 25 active full-stack apps; the backend docs and local FuseBase CLI instructions show Node backend jobs/secrets. But the vendor docs say backend apps cost more and the 20-code evidence does not explicitly establish no incremental runtime bill for THIS account. **Do not deploy/activate a FuseBase backend until the account's actual billing UI or written vendor terms confirm $0 incremental cost.** It also requires porting ~the entire Python/Plaid backend to Node.

## Candidate C: Supabase Edge Functions + Supabase Cron — possible orchestrator, unlikely full sync

Supabase Free includes 500k Edge Function invocations/month and Postgres Cron can call functions, but Free Edge Functions cap at 2s active CPU, 150s wall time, 256 MB memory. The current database has `pg_cron`/`pg_net` extensions absent (confirmed via read-only SQL), so scheduler plumbing must be enabled/installed/tested. Supabase Free projects pause after 7 days of low user DB activity; a scheduled `pg_cron` internal job alone may not count as user traffic. Need confirm keepalive semantics. Functions can host small API/webhook handlers or orchestrate chunks, but require the same TypeScript port and finite per-call limits. Do not rely on a cron trigger keeping the DB active.

## Candidate D: GitHub Actions — free scheduler only if public repo, not the Plaid API host

GitHub-hosted runners are free for public repositories; usage for private repos depends on plan. Scheduled jobs can run the incremental sync but do not provide an always-on API endpoint for Link redirects/webhooks. Public repositories' scheduled workflows may disable after 60 days of inactivity. Publishing the fork creates a public-source/privacy tradeoff (no secrets/data in Git); evaluate before use.

## Rejected: Render for the $0-only rule

Render Free Web Services spin down after 15 minutes, receive 750 instance hours/month, and external API traffic may trigger suspension. Render Cron Jobs are not free and have a $1/month minimum. Therefore, do not use Render for this project while the $0 rule is in force.

## Decision

Cloudflare Workers Free is the best candidate because the user already has Cloudflare, it supports FastAPI/Python with a 10ms CPU cap, and the GitHub base can be adapted. The next engineering task is a narrowly scoped test branch: prove one async Plaid Sandbox link-token call, one webhook signature validation, and one small data-sync page in the deployed Worker, then inspect CPU. Keep FuseBase frontend + Supabase Free data/Auth. The complete live product is **not yet connected to Plaid**.

## Change log

2026-11-17 — Initial $0-only alternatives review: read official Cloudflare/Supabase/Render docs, inspected account plan note and upstream requirements, locally confirmed FastAPI boot and imported Plaid generated code after experimental vendoring; no hosted Cloudflare deployment was made. New test branch/task needed for actual async/non-threaded port and remote validation.
