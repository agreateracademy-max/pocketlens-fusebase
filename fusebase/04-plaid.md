# 04 — Plaid

Allowlist the sync service in the Plaid dashboard:

- Redirect URI: `https://<sync-service>.onrender.com/link`
- Webhook: `https://<sync-service>.onrender.com/webhook/plaid`

Set the same values as `PLAID_REDIRECT_URI` and `PLAID_WEBHOOK_URL` on the API service and
redeploy.

Two credential modes, usable together:

- Server-wide: `PLAID_CLIENT_ID` / `PLAID_SECRET` / `PLAID_ENV` on the sync service.
- Per user: each user adds their own Plaid account in Settings; the secret is validated and
  stored Fernet-encrypted in `plaid_credentials`.

Without the webhook the daily cron still syncs. With it, updates are real-time.
