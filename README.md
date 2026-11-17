# ship-check

A Claude Code skill that checks if your AI-built app is ready for production.
It reads your project and scores 10 points: backups (and a tested restore), secrets in the code, CI, safe mode for
messages and payments, fake data in dev, migrations, after-deploy checks, rollback and alerts.
It **changes nothing** until you choose a fix.

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

ثبّتها، حلّ Claude Code في الـ projet متاعك واكتب `/ship-check`.
تقرا الـ projet متاعك، تعطيك 10 نقاط بـ PASS / WARN / FAIL، وما تبدّل حتى شي كان كي تقلها إنت.

## Why

My startup had no database backup until the day I prepared the repo for new developers.
These are the 10 things we check now before every deploy at reservations.tn.

— Baha Eddin Mselmi
