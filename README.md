# ship-check

A Claude Code skill that checks if your AI-built app is ready for production.
It reads your project, runs 15 checks and gives you a **Ship Score out of 100**:

- **Can you lose data or leak it?** backups and a tested restore · users seeing each other's data (server checks,
  Supabase RLS, Firebase rules) · secrets in the code, the git history or the browser
- **Can you lose money or customers?** payments and webhooks · AI/API bills · emails/SMS sent from dev · fake data in dev ·
  error leaks · fake or vulnerable packages · upload limits
- **Can you ship and recover calmly?** no coding on the server · CI · migrations · after-deploy check · rollback and alerts

Works with Supabase, Firebase, Vercel, Netlify, or your own server. It **changes nothing** until you choose a fix.
It is not a full security audit.

## Install (1 minute)

```bash
git clone https://github.com/bahaeddinmselmi/ship-check.git
mkdir -p ~/.claude/skills
cp -r ship-check/skills/ship-check ~/.claude/skills/
```

Then open Claude Code in your project and type:

```
/ship-check
```

or just ask: « is my app ready for production? »

Other agents that read `SKILL.md` files (Codex, Cursor…) can use `skills/ship-check/SKILL.md` too.

## نسخة تونسي

ثبّتو، حلّ Claude Code في الـ projet متاعك واكتب `/ship-check`.
يقرا الـ projet متاعك، يعدّي 15 test ويعطيك Ship Score على 100، وما يبدّل حتى شي كان كي تقلّو إنت.

## Why

My startup had no database backup until the day I prepared the repo for new developers.
These are the 10 things we check now before every deploy at reservations.tn.

— Baha Eddin Mselmi
