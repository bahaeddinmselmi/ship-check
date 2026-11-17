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
