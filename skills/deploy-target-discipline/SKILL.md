---
name: deploy-target-discipline
description: Procedure for deploying, debugging "reverted/stale" production, or answering "where does this actually deploy". Use BEFORE running any deploy command (vercel/railway/fly/render/netlify/supabase push), when prod looks old or reverted, when a repo has multiple platform configs, or when wiring an app to a hosted backend. Finds the one canonical target, never deploys local state by hand, and verifies with a route only the new code has.
---

# Deploy-Target Discipline

A repo's deploy configs lie about what production is. One CRM I work on carries **four** platform configs (`railway.toml`, `render.yaml`, `fly.toml`, plus Vercel/Netlify dashboard integrations) — exactly one is canonical. A dashboard "kept reverting to an old version" for weeks; the code on main was fine all along — the URL pointed at a duplicate project wired to a stale repo, and every real build was silently erroring on an invalid `vercel.json` key. **The failure mode is never "git lost my code." It's always "which target am I actually looking at, and did the build actually run."**

**Keywords**: deploy, reverted, stale prod, old version, vercel, railway, fly, render, netlify, supabase push, canonical, which URL, env vars, production.

## The Rules

1. **One canonical target per app; find it before touching anything.** Configs present in the repo ≠ live. Check which platform's GitHub integration is connected and which URL the user actually visits. Write the answer down (memory) — this question recurs.

2. **Never deploy by hand when a git integration exists.** `railway up` / `vercel --prod` / `fly deploy` ships your *local working tree*, not the merged commit — that's how multi-platform drift starts. The deploy path is: merge to main → platform auto-deploys. Hand-deploys only for repos with no integration, and say so when you do.

3. **Verify with a route only the new code has.** `curl` the prod URL's healthcheck **plus** an endpoint/string that didn't exist before this deploy. A 200 on `/healthz` proves the old version is up, not the new one.

## When prod "reverted" or looks stale — diagnose in this order

Each of these has actually been the cause; none of them is a git problem:

1. **Wrong project/URL.** Duplicate platform projects (one wired to a stale or wrongly-named repo) — the user views the duplicate. Enumerate the account's projects and check which repo each is git-connected to.
2. **Builds silently failing.** An invalid config key (`"public": true` in `vercel.json`) errors *every* build, so prod pins to the last good deploy and looks "reverted." Read the platform's build log for the latest commit before any other theory.
3. **Cache.** `Cache-Control: max-age=3600` shows stale up to an hour — hard-refresh (Cmd+Shift+R) before declaring a deploy missing.
4. **Dead CI you assumed was deploying.** GitHub Actions can be billing-blocked (spending limit — the runner never starts). If Actions is the assumed deploy path, confirm the run actually executed. Note: platform-native integrations keep working when Actions is dead.

## Environment-variable traps

- **Build-time vs runtime.** `NEXT_PUBLIC_*` / `VITE_*` are inlined at **build** — setting them requires a rebuild, and a missing one means the shipped bundle points at `127.0.0.1` for every real visitor. After setting, verify the deployed bundle actually references the new value.
- **Re-pointing a backend** (e.g. staging Supabase → real project) means updating env on *every* consuming service — API server *and* web build — then E2E: sign in against prod, hit an authed route, confirm the response comes from the new backend.
- **Frontend must be same-origin** with the backend that serves it; hardcoded `localhost:PORT` or old-platform hostnames in shipped pages die silently. After converting to same-origin, check the backend actually implements those routes — same-origin 404s are the follow-up gap.

## Permission ceilings (hosted backends)

Non-owner org membership splits capabilities: DB-password-gated commands work (`supabase db push`, `migration list`), but Management-API settings changes **403** (`supabase config push`). Consequence: hosted Auth config can silently diverge from repo `config.toml` (OTP length, confirmations, MFA). Don't burn time retrying — record the divergence, name the owner action needed, move on.

## Reporting

- State the canonical target and how you confirmed it ("Railway is prod: git-connected, serves both API and static frontend; render/fly configs are stale secondaries").
- After a deploy: the verifying evidence — new-code route + response, not just "deployed successfully."
- Route any code changes involved through [[ship-gate]].

## Real cases

- **An internal analytics dashboard** — "reverting" = duplicate Vercel project on a stale repo + invalid `vercel.json` failing every build + 1h cache. Three causes, zero git problems.
- **A CRM** — 4 deploy targets configured, Railway canonical (serves frontend too); redundant Vercel projects git-disconnected; ~37 hardcoded localhost/fly/netlify refs converted to same-origin, exposing 5 routes the Railway server doesn't implement yet.
- **A web app** — missing `NEXT_PUBLIC_SUPABASE_URL` meant the live signup form POSTed to 127.0.0.1 for real visitors; fixed via `railway variables --set` + rebuild, verified by Playwright on the live page.
- **A fintech app** — staging→production project migration: re-link, `db push`, re-point both Railway services' env, E2E verify. Same non-owner 403 ceiling.
