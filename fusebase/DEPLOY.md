# Deploy PocketLens to FuseBase

Target: Sans Titre org `u25ynk` (`sanstitre.nimbusweb.me`).

Run in order:

1. `01-supabase.md` — project, migrations, redirect URLs, SMTP.
2. `02-sync-service.md` — Render blueprint for the Python Plaid service.
3. `03-fusebase-app.md` — scaffold, copy the SPA, build, deploy.
4. `04-plaid.md` — redirect URI, webhook, credentials.
5. `05-verify.md` — end-to-end checks.

Env: `env.fusebase.example`.

App create and deploy are CLI-only. The FuseBase MCP exposes portals, dashboards, Web
Studio and isolated stores, but no app-create or app-deploy operation.
