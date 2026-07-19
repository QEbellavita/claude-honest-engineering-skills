---
name: security-triage
description: Procedure for triaging a large set of security findings (from a scanner, audit, or sweep) down to confirmed, ranked, fixable issues. Use when facing many vulnerability findings, running a security sweep, reviewing auth/tenant-isolation, or hardening an app before go-live. Dedupes, confirms by reproduction, sweeps BOLA/IDOR routes, and treats RLS as the last line of defense — not the only one.
---

# Security-Triage

A scanner's finding count is noise until triaged — a 118-finding dump collapsed to **43 confirmed** once each was reproduced. Untriaged findings waste effort on false positives and bury the real P0s. This procedure turns a pile into a ranked, confirmed, fixable list.

**Keywords**: security findings, vulnerability, triage, sweep, hardening, BOLA, IDOR, auth, RLS, tenant isolation, pentest, P0.

## The Rule

**Confirm by reproduction before you fix.** Do not trust the scanner's severity or its existence claim. Reproduce the exploit (or prove it can't happen) — that separates the 43 real from the 75 phantom.

## Checklist (create one todo per item)

1. **Dedupe and cluster.** Collapse findings that are the same root cause across many routes into one issue with N sites. One missing auth middleware is one fix, not twenty findings.

2. **Confirm each by reproduction.** For each cluster, actually trigger it — unauthenticated request, swapped ID, forged token. Mark **CONFIRMED** (reproduced), **FALSE-POSITIVE** (can't reproduce / already mitigated), or **NEEDS-INFO**. Only CONFIRMED proceeds.

3. **Sweep BOLA / IDOR across every route.** The highest-yield class. For every endpoint that takes a resource ID, prove the query is scoped to the caller — can user A read/mutate user B's object by changing the ID? Sweep systematically; these cluster (10+ across a handful of route files in one sweep).

4. **Check the auth-boundary edge cases** the scanner misses:
   - Auto-enroll / sync paths that skip the auth check.
   - "Soft auth" that proceeds when auth is absent instead of rejecting.
   - Background/webhook/service paths using a service-role client where a user-scoped one belongs.

5. **Rank by real impact.** P0 = auth bypass, cross-tenant data access, RCE, secret exposure. Order the confirmed list most-severe first. Fix in that order.

6. **RLS is the last line of defense, not the only one.** Fix the app-layer check *and* ensure the data layer would still contain the breach if the app check regresses — user-scoped DB client (`userScopedClient`) so row-level security enforces isolation even when app logic is wrong. Never rely on RLS alone; never rely on app-layer alone.

7. **Add the regression test.** Every confirmed fix gets a test (cross-tenant IDOR test, unauthenticated-request test) so it can't silently return. Route the whole fix through [[ship-gate]].

## Reporting

- Report the funnel honestly: "118 findings → 43 confirmed → N P0." Don't quote the raw scanner count as if all are real.
- For each P0: the repro, the fix, the regression test.
- Note deferred items explicitly — nothing silently dropped.

## Real cases

- **P0 sweep** — 118 findings triaged to 43 confirmed; shipped 4 P0 auth fixes + GDPR erasure + Apple-ID collision + 10 T1 BOLA across 6 routes (one PR, full suite green). Adversarial re-review found 3 broken paths (watch/sync auto-enroll, emotion-memory softAuth, social-contagion) that were then fixed.
- **A bank-grade bar** — AU banking/money data: build "overboard," RLS as backstop, MFA, immutable audit log, headers/rate-limit, compliance scoping.
