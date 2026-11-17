# ship-check — the 15 checks

## Stack detection

| Detect | Look for |
|---|---|
| Framework | `next.config.*`, `vite.config.*`, `nuxt.config.*`, `astro.config.*`, `app.json` (Expo), `manage.py`, `go.mod` |
| Database / BaaS | `supabase/`, `@supabase/*`, `firebase.json`, `firestore.rules`, `prisma/`, `drizzle.config.*`, `DATABASE_URL` in `.env.example` |
| Hosting | `vercel.json`, `netlify.toml`, `fly.toml`, `railway.*`, `render.yaml`, `Dockerfile`, `docker-compose*.yml`, deploy scripts |
| Payments | `stripe`, `@paddle/*`, `lemonsqueezy`, local gateways; webhook routes |
| Messages | `resend`, `nodemailer`, `@sendgrid/*`, `twilio`, WhatsApp APIs, push (`firebase-admin`, `web-push`, `expo-server-sdk`) |
| AI APIs | `openai`, `@anthropic-ai/sdk`, `@google/genai`, `ai` (Vercel AI SDK), OpenRouter |
| Uploads | `multer`, `formidable`, S3/R2/Supabase Storage clients |

## Tier A — you can lose data or leak it (FAIL = 12 points each)

**A1. Backups exist, off the server, and a restore was tested.**
Self-hosted DB: a scheduled dump (`pg_dump`/`mysqldump`/cron/systemd timer) with retention, uploads included, and a copy
somewhere else (S3/R2/B2/another server). Managed: check the provider actually backs up on the user's plan (e.g.
Supabase daily backups and point-in-time restore depend on the plan; Firebase needs scheduled exports or PITR enabled)
— ask if unknown. Never restored = WARN at best: propose restoring into a throwaway local database and comparing row counts.

**A2. Users can only reach their own data.**
Every server route / server action / API that reads or changes data checks *who* is asking and *that they own the
record* — not only that they are logged in. Look for handlers taking an `id` from the URL/body and querying by it alone.
Supabase: RLS enabled with policies on every table exposed to the anon/authenticated key (`supabase/migrations`,
`alter table … enable row level security`); service-role key only on the server. Firebase: `firestore.rules`/
`storage.rules` not `allow read, write: if true`. Admin routes protected server-side, not just hidden in the UI.

**A3. No secrets in the code, the git history, or the browser.**
Tracked files and history: `git grep -nE "(sk_live_|sk-proj-|sk-ant-|AKIA[0-9A-Z]{16}|ghp_|xox[bp]-|-----BEGIN [A-Z ]*PRIVATE KEY)"`,
committed `.env*` files other than examples (`git ls-files | grep -E "\.env"`), `git log --all -p -S '<found key prefix>'`.
`.env` in `.gitignore`. Browser bundles: no server secret behind a public prefix (`NEXT_PUBLIC_`, `VITE_`, `EXPO_PUBLIC_`,
`PUBLIC_`, `REACT_APP_`) — public ones may hold only keys designed to be public (Supabase anon key *with RLS*,
Stripe publishable key). A real leak = FAIL + rotate.

## Tier B — you can lose money or customers (FAIL = 6 points each)

**B1. Payments are trusted from the server only.** (FAIL = 8 points) Prices/amounts computed server-side, never taken from the client.
Payment webhooks verify the provider's signature (or re-fetch the payment from the provider) and are idempotent
(the same event twice doesn't credit/ship twice).

**B2. Paid APIs can't run up your bill.** AI and other metered APIs called only from the server, behind auth or a rate
limit, with a max tokens/size per request; a spending limit set in the provider dashboard (ask). Public endpoints that
send email/SMS or call AI have rate limiting.

**B3. Safe mode outside production.** Code that sends emails, SMS, WhatsApp, push or charges cards can't really fire from
dev/test (env flag, sandbox keys, log-only). Check every send/charge call site for the guard.

**B4. Fake data in development.** Seed script or fixtures; no copy of production customer data on laptops.

**B5. Errors don't leak, and the app survives a failing service.** No stack traces/SQL errors returned to users in
production, no `console.log` of tokens or personal data; calls to external services have a timeout and a fallback
instead of a blank page.

**B6. Dependencies are real and patched.** Every package in the manifest exists and is the intended one (AI can invent
names — check unfamiliar ones on the registry); run the ecosystem's audit (`npm audit --omit=dev`, `pip-audit`…) if
available and report critical/high only. Lockfile committed.

**B7. Uploads are limited.** Max size and allowed types checked server-side; uploaded files not served as executable HTML/JS.

## Tier C — you won't be able to ship or recover calmly (FAIL = 4 points each)

**C1. No coding on production.** Branch → review/PR → deploy. FAIL if the deploy is editing files on the server or the only
copy of the code lives there.

**C2. Machine checks before merge.** CI runs type check/lint/build and tests on every PR (or the host's build blocks a broken
deploy). WARN if there are no tests on money, auth or data-changing logic.

**C3. Database changes through migrations.** Versioned migration files, reviewed; no manual edits of production tables.

**C4. After-deploy check.** A health endpoint or a written smoke test (open the site, do one real action, read logs), ideally
run automatically at the end of the deploy.

**C5. Rollback + alerts.** The previous version can go back with one command (platform rollback, previous image/tag);
an uptime monitor on the public URL, error tracking (Sentry or similar), disk-space alert on a VPS.

Points: Tier A 3×12 = 36 · Tier B 8 + 6×6 = 44 · Tier C 5×4 = 20 · total 100.
