---
name: ship-check
description: Check whether an app built with AI is ready for production — backups (and a tested restore), secrets in the code, CI checks, tests, safe mode for messages/payments, fake data in dev, migrations, post-deploy checks, rollback and uptime alerts. Use when the user asks "is my app ready for production?", "ship check", "deploy check", "what am I missing before launch", or before a first deploy.
---

# ship-check

You audit the user's project against 10 points that break real apps after launch. You are **read-only** until the user
says which fix they want: never delete, rotate, push, deploy or touch a production server or database on your own.

## Steps

1. **Understand the project first.** Read the README, package/dependency files, Dockerfile/compose, CI folders
   (`.github/workflows`, `.gitlab-ci.yml`), deploy scripts and the database setup. Note the stack, where it runs
   (VPS, Vercel, Netlify, Railway…), and the database. Ask the user only what the files can't tell you
   (e.g. "where is the production database hosted?").
2. **Check each point below.** For each: `PASS`, `WARN` or `FAIL`, with the evidence (file:line or "not found")
   and the smallest fix for *this* stack.
3. **Report** as a table (point · status · evidence · fix), then the **top 3 fixes in order**, starting with anything
   that can lose data or leak secrets.
4. **Offer to fix one point at a time.** Before any change, say exactly what you will change. Never run a restore,
   migration or deploy against production; use a local or test database.

## The 10 points

1. **Backups, tested.** Automatic database backup (cron, managed-DB backups, `pg_dump`/`mysqldump` script) with
   retention; uploaded files backed up too; at least one copy **off the server**. A backup never restored is a `WARN`
   at best: propose restoring into a throwaway local/test database and comparing row counts.
   Managed DB (Supabase, Neon, RDS, PlanetScale…): check the plan actually includes backups / point-in-time restore.
2. **No coding on production.** Changes go branch → PR/review → deploy. `FAIL` if the deploy is "edit files on the
   server" or the only copy of the code lives there.
3. **Machine checks before merge.** CI runs type check / lint / build and tests on every PR. `WARN` if CI exists but
   tests are absent; `FAIL` if nothing runs.
4. **Safe mode outside production.** Code that sends emails, SMS, WhatsApp, push notifications or charges payments
   must not really send from a dev/test environment (env flag, sandbox keys, or log-only). Grep the send/charge call
   sites and check for a guard. `FAIL` if a local run could message real customers or charge real cards.
5. **Fake data in development.** A seed script or fixtures; no copy of production customer data on laptops.
6. **Secrets out of the code.** Search the tracked files and git history for keys, tokens, passwords and connection
   strings (e.g. `git grep -nE "(sk_live|sk-|AKIA|api[_-]?key|secret|password|token)\s*[:=]"`, committed `.env`
   files, `git log -p -S` for anything found). `.env` in `.gitignore`. A leaked key = `FAIL`: tell the user to
   **rotate it** (deleting the line doesn't help, it stays in history). Never print a secret's full value.
7. **Database changes through migrations.** Schema changes are versioned migration files, reviewed, one per PR;
   no manual edits of production tables.
8. **After-deploy check.** A health endpoint or a written smoke test (open the site, do one real action, read logs).
9. **Rollback.** The previous version can be put back with one command (previous image/tag, platform rollback,
   `git revert` + redeploy). `WARN` if nobody knows how.
10. **Alerts before customers notice.** Uptime monitor on the public URL, error tracking (Sentry or similar),
    disk-space alert on a VPS.

## Tone
Plain words, short sentences. The user may be a beginner who built the app with AI: explain *why* each point
matters in one line, with a concrete failure ("if the disk dies tonight, every customer and booking is gone").
Reply in the user's language.

---
Made by Baha Eddin Mselmi (Tunis) — from what we run in production at reservations.tn.
