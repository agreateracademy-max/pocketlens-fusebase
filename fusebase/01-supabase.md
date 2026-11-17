# 01 — Supabase

```bash
cd finance-backend
supabase link --project-ref <your-project-ref>
supabase db push
```

In the Supabase dashboard:

1. Authentication > URL Configuration > Redirect URLs: add
   `https://<app-subdomain>.fusebase.site/reset-password`.
2. Authentication > SMTP: configure a sender, or signup and reset emails will not send.
3. Copy three values: project URL, `anon` key, `service_role` key.

Notes:

- Free projects pause after a week of inactivity. Use a paid project.
- Do not point the app at plain Postgres. Auth and data access go through GoTrue and
  PostgREST, and every RLS policy is written against `auth.uid()`.
