# last-90 — finish the last 90% of a web app launch

> The first 10% is the working demo agents nail. **The last 90%** — authorization,
> monitoring, legal, backups, abuse limits, CI/CD, an ops runbook — is what separates
> a demo from something safe to put in front of real users. This skill does that
> work, on a loop, until your app is genuinely ship-safe.

A goal-driven [Claude](https://claude.com/claude-code) skill. Point it at a freshly
deployed web app and it scores production-readiness, then **does every fix it can on
its own**, only stopping to ask you for the things it can't get without you (an
error-tracker DSN, GitHub Actions access, a backup bucket, a legal decision).

Open source first: monitoring recommendations default to self-hostable tools —
[Bugsink](https://www.bugsink.com/) or [GlitchTip](https://glitchtip.com/) for error
tracking (both speak the Sentry SDK protocol, so hosted Sentry is a drop-in swap) and
[Uptime Kuma](https://github.com/louislam/uptime-kuma) or
[Gatus](https://github.com/TwiN/gatus) for uptime. Nothing in the gate forces a paid
SaaS.

It's the follow-on to a deploy: **ship → `/last-90` → confident it's set up properly.**

## What it checks — "the scale"

A **Launch Gate** of 10 binary blockers (target **10/10**) on top of **11 graded
dimensions**. Every item is tagged 🤖 *agent does it* or 🙋 *needs your account/decision*.

| Launch Gate (must all pass) |
|---|
| Core flows work against the prod build · Real auth (vetted lib) · Authorization / tenant isolation · Security hardening (validation, rate limits, no secrets in repo) · Legal pages · Error monitoring live in prod · UGC moderation *(if applicable)* · CI/CD on every PR · Health check endpoint · Backups **with a rehearsed restore** |

Dimensions go deeper: MVP audit, security, failure modes, unit economics,
scalability, observability, data protection, UX resilience, legal & risk, abuse
prevention, ops playbook. Full criteria in
[`skills/last-90/references/scale.md`](skills/last-90/references/scale.md).

Stack-aware (works for VPS+SQLite+Litestream *or* Vercel+Supabase+Sentry), not locked.

## How it runs

1. **Detects your stack** from the repo.
2. **Scores a baseline** and splits the gaps into an autonomous queue and a needs-you queue.
3. **Access & Connections Intake** — one upfront question: which services can you grant now vs defer?
4. **Loops autonomously** — implements and verifies every 🤖 item.
5. **Batches one check-in** for the 🙋 items, with exact steps. Resumes when you grant access.
6. **Verdict:** SHIP-SAFE / NOT-YET, with evidence.

## Install

**As a plugin (one command):**
```
/plugin marketplace add Avanderheyde/webapp-last-90
/plugin install last-90@webapp-last-90
```

**As a plain skill:** copy `skills/last-90/` into `~/.claude/skills/`.

## Use

From inside your deployed app's repo:
```
/last-90              # score + finish the last 90%
/last-90 explain      # same, but teaches you what each item is and why it matters
```

`explain` mode is for first-time vibe coders — it narrates *what* it's doing and
*why it matters* (the concrete failure each fix prevents) as it goes.

## License

MIT
