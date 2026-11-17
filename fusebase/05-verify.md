# 05 — Verify

1. Open the FuseBase app URL: the logged-out demo dashboard renders.
2. Sign up, confirm the email arrives, log in.
3. Settings > add Plaid credentials > link a sandbox institution.
4. Confirm transactions appear after the first sync; trigger a manual sync and check the
   Render API logs for a 200.
5. Confirm the scheduler ran `sync.py` at least once.
6. Optional: embed in a portal via the customizer (Add block > Fusebase App) or via MCP
   `addPortalAppBlock` using `productId` + `appId` from `findPublishedApps`.

Never place `SUPABASE_SERVICE_ROLE_KEY`, `CREDENTIALS_ENC_KEY` or Plaid secrets in the
FuseBase app env. They belong only on the sync service.
