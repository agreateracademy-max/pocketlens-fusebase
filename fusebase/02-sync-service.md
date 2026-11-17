# 02 — Sync service (Render)

Python/FastAPI cannot run on FuseBase app backends, so this stays on Render.

1. Render > New > Blueprint > `agreateracademy-max/pocketlens-fusebase`.
2. API service variables: `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY`,
   `CREDENTIALS_ENC_KEY`, `WEB_ORIGINS`, `PLAID_WEBHOOK_URL`, `PLAID_REDIRECT_URI`.
3. Scheduler cron: the same `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` and
   `CREDENTIALS_ENC_KEY`.

Generate the crypto key once:

```bash
python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

`CREDENTIALS_ENC_KEY` must be byte-identical on API and scheduler. Losing or changing it
orphans every stored Plaid secret.

`WEB_ORIGINS` is the FuseBase app origin, scheme included, no trailing slash. Without it
`POST /sync/trigger` fails CORS.
