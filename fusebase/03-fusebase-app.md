# 03 — FuseBase app (LIVE)

The app is already registered and deployed in the Sans Titre FuseBase organization.

- Product: `PocketLens` (`xmxwx6qkgetfavfm`)
- App: `PocketLens` (`7pgixih7yyu9ahyz`)
- URL: https://pocketlens.thefusebase.app/
- Access: visitor, organization member, manager and owner

Verification: opened the live URL in an authenticated Chrome session. The React SPA rendered
its full dashboard (Home, Transactions, Budget, Explore, Balances, Settings), demo data, and
no error screen. The build has `VITE_SUPABASE_URL` and public anon key configured; backend
URL currently points at Render placeholder `https://pocketlens-api.onrender.com` until the
Python sync API is deployed.

Source in `apps/pocketlens/` came from upstream `web/`, with the FuseBase scaffold's CLI-managed
`package.json`/Vite settings retained and PocketLens dependencies merged in. Use the CLI for
future edits/deploys:

```bash
fusebase dev start
fusebase deploy --app pocketlens
fusebase app list
```
