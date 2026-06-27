# The Last 90 — the scale

The first 10% (a working app) is what agents do well. The **last 90%** is the
unglamorous production work everyone skips: authz, monitoring, legal, backups,
abuse limits, ops. This scale grades that work so a freshly-deployed web app can
be made *actually safe and set up properly* before real users arrive.

Two layers:

1. **Launch Gate** — 10 binary blockers. Each is PASS or FAIL. You do not ship
   to real users until this is **10/10** (or an item is explicitly deferred by
   the owner with eyes open).
2. **Dimensions** — 11 graded areas (the depth behind the gate). Each gets
   `✅ Solid` / `🟡 Partial` / `❌ Missing` / `⏭️ Deferred`.

Each criterion is tagged:
- 🤖 **Autonomous** — the agent can do it from the code/repo alone. Just do it.
- 🙋 **Needs-human** — requires an account, credential, secret, paid tier, or a
  product/legal decision. Batch these for a single check-in; never guess them.

---

## Launch Gate (10 binary blockers)

| # | Blocker | PASS bar | Tag |
|---|---|---|---|
| 1 | **Core flows work** | Signup → primary action → result works against the **prod build** (`next build && start`, not just dev). | 🤖 |
| 2 | **Auth is real** | A vetted library (Auth.js v5 / Supabase Auth) — never hand-rolled. Session validated on every protected route + server action. Logout works. | 🤖 (code) / 🙋 (OAuth client IDs) |
| 3 | **Authorization / tenant isolation** | Supabase: RLS enabled on **every** table. SQLite/levels.io: every table has an owner column and every query filters by the authed user, centralized so it can't be forgotten. Proven: user A cannot read user B's row. | 🤖 |
| 4 | **Security hardening** | Server-side input validation on all writes; rate limiting on all mutations; SSRF guard if the app fetches user-supplied URLs; **no secrets in the repo** (`.env` gitignored, `.env.example` present). | 🤖 |
| 5 | **Legal pages** | Privacy Policy + Terms of Service published and linked in the footer; cookie/consent notice if you set non-essential cookies; a real contact address. | 🤖 (draft) / 🙋 (contact + approval) |
| 6 | **Error monitoring** | Sentry (or equivalent) live in **production**, plus a global error boundary that reports and shows a recovery UI. | 🤖 (wiring) / 🙋 (DSN) |
| 7 | **UGC moderation** *(only if the app accepts user-generated content)* | Report/flag mechanism on user content + a way to action reports. N/A if no UGC — mark ⏭️ N/A. | 🤖 |
| 8 | **CI/CD** | CI runs build + tests on every PR/push so a broken commit can't reach prod. | 🤖 (workflow file) / 🙋 (enable Actions + secrets) |
| 9 | **Health check** | `/api/health` (or `/healthz`) that pings the DB and returns 200/503. | 🤖 |
| 10 | **Backup + restore tested** | Automated backups running (Litestream→R2, or Supabase PITR) **and a restore actually rehearsed once.** Backups you've never restored don't count. | 🤖 (config) / 🙋 (bucket/paid tier + restore drill) |

**Headline score = N/10.** Anything below 10 is not ship-safe.

---

## Dimensions (graded depth)

For each: state the grade, one line of evidence, and the single highest-value
next action.

**0. MVP audit** — Stack, dependency count, table/route inventory, env config is
clean (no hardcoded URLs, 3-tier local/preview/prod). 🤖

**1. Security** — auth, authz, validation, SSRF, rate limiting, API-key hashing,
security headers (CSP/HSTS/X-Frame/X-Content-Type/Referrer/Permissions). Note
gaps: CSRF posture, key rotation, CSP report-uri. 🤖

**2. Failure modes** — What happens when the DB is down, a token expires, a rate
limit hits, a payload is huge, an unhandled error throws? Graceful degradation +
retry where cheap. 🤖

**3. Unit economics** — Monthly cost at launch and at 1K/10K/100K users; where
the first bottleneck/bill spike is (usually no caching → every request hits the
DB). 🤖 (model) / 🙋 (pricing decision)

**4. Scalability** — No unbounded queries (e.g. fetching 10K rows to compute one
rank → use a window function); pagination on feeds/comments; ranking/heavy work
in SQL not JS; `next/image` over raw `<img>`. 🤖

**5. Observability** — Error tracking + **uptime monitoring pinging /health** +
alert rules + structured logging. 🤖 (logging) / 🙋 (uptime account + alerts)

**6. Data protection** — Encryption at rest, no PII in logs, account deletion
(GDPR erasure), data export (portability), backup tested. 🤖 (most) / 🙋 (DPA/legal)

**7. UX resilience** — Loading/skeleton states, error/empty states, client-side
validation feedback, server-side search if the list grows, mobile layout. 🤖

**8. Legal & risk** — Privacy + ToS + consent (gate items) plus DMCA/takedown
path and content-ownership terms if you host user content. 🤖 (draft) / 🙋 (review)

**9. Abuse prevention** — Rate limits, auth gates, validation, report mechanism,
spam/profanity filtering, vote/action-manipulation detection, IP-based limits on
public endpoints. 🤖

**10. Operations playbook** — Deploy checklist, rollback procedure, incident
response notes, runbook in `CLAUDE.md`/`README`. For levels.io/VPS: confirm the
box comes back after reboot (systemd brings up app + Caddy + Litestream). 🤖

---

## Verdict rule

Report **SHIP-SAFE** only when: Launch Gate = 10/10 (N/A items excused) **and**
no dimension is `❌ Missing`. `🟡 Partial` is allowed only if the owner has seen
it and accepted the residual risk. `⏭️ Deferred` requires an explicit owner
decision, recorded in the report with the reason.
