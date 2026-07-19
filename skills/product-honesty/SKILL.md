---
name: product-honesty
description: Scoping discipline for deciding whether a product claim or feature direction is honestly buildable on the real substrate — before you promise it, design toward it, or put it in the pitch. Use when framing a north star, scoping a feature, writing user-facing capability copy, or choosing what to build next. Catches sci-fi framing over placeholder cores, claims the data can't support, and "engineering will fix it" hand-waving over founder-only bottlenecks.
---

# Product-Honesty

The most expensive mistake isn't a bug — it's building toward a claim the substrate can't honestly support. A 30-engine "emergent mind" that's actually a random-noise observation matrix over 0-row longitudinal tables *looks* like a product and delivers nothing real. This skill is the scoping gate that keeps the product claim tied to what's actually there. It completes the honesty triad: [[stub-detection-audit]] asks *is the code real*, [[honest-negatives-validation]] asks *is the result real*, and this asks **is the product claim honest and buildable**.

**Keywords**: north star, product direction, feature scope, capability claim, pitch, roadmap, buildable, sci-fi, real vs fabricated, feasibility, what to build next, honest framing, wedge.

## The Rule

**A capability claim is a liability until it traces to a real asset and survives the honest-framing test.** Before you promise it, design toward it, or write the copy: name the real, verified thing it stands on. If the honest answer is "a placeholder we intend to light up," the claim is fiction and lighting up the placeholder deepens the debt, not the product.

## Checklist (create one todo per item)

1. **List the real assets — the allowlist you're allowed to build on.** What actually works, verified by reading the leaf code (not the structure)? Build the claim on ONLY these. In this ecosystem the real set is small and known: per-signal `user_baselines` with a calibration gate, real HealthKit signals (guarded `source !== 'simulated'`), an honest linear signal→state mapping that returns `null` not a fake number, persisted Big Five, a glass-box dispute/correct/delete user model, real DSAR erasure. Everything else is a candidate for step 2.

2. **Name the fabricated cores you must NOT build on.** Which "capabilities" are placeholder — random-init matrices never trained, stateless 0.5-defaults, dead fusion from a shape-mismatch bug, 0-row tables? Lighting these up looks like progress and isn't. Run [[stub-detection-audit]] on anything you're unsure of before it enters the plan.

3. **Frame to what the data honestly supports — and what's defensible.** Claim the thing the real assets actually deliver (physiological *state* — recovery, dysregulation — from real baselines), not the thing that sounds better but the data can't back (emotion recognition, "it knows how you feel"). Honest framing is also often the regulatory shield (state-not-emotion sidesteps the EU AI Act emotion-inference rules). If the honest claim is narrower, ship the narrower claim.

4. **Make "real-or-honestly-absent" the moat, not a limitation.** The differentiator is the posture incumbents won't adopt: abstain when uncertain, show your work (corrigible glass box), let the user delete, don't lock honesty behind a subscription. A magic insight is easy to fake and easy to copy; honesty on real longitudinal data is the defensible position. Design the abstain path as a feature, not an apology.

5. **Solve cold-start honestly.** The tempting fix for "no data on day 1" is fabrication (seed a fake baseline). The honest unlock is real retrospective data — e.g. HealthKit historical backfill crosses the calibration gate on day 1 from weeks already on the user's watch. Prefer the unlock that makes the day-1 claim *true* over the one that makes it *look* true.

6. **Separate code-fixable from founder-only bottlenecks — don't paper over them.** Be brutally honest about what actually gates the product. If distribution (TestFlight broken, Individual not Org account, never submitted to review) and durable storage (prod DB wiped every redeploy) are what stand between you and real users, then no amount of feature-building moves the needle — and a plan that implies engineering will fix them is dishonest. Name the non-engineering blocker as the blocker.

7. **Right-size the target metric to honest value.** Don't inflate the goal to sound ambitious. 20–50 real users × durable weeks of honest data beats 10,000 × ephemeral. Pick the metric that measures the real thing (durable retrospective depth), not the vanity number.

## Reporting

- State the claim and its foundation in one line: "wedge = 'Your Baseline, from Day 1' — stands on `user_baselines` + HealthKit backfill, both real; does NOT claim emotion recognition."
- Explicitly list what you're KILLING and why it's not honestly buildable — a decision, not a silent omission.
- Flag any founder-only / non-engineering bottleneck separately from the build plan, so it can't be mistaken for something code will solve.
- If the honest feasibility is low, say the number and where the hardness actually lives (usually distribution, not code).

## Real cases

- **A north-star reframe** — killed the "90-second reset coach" *and* the sci-fi "fused emergent mind" (digital twin = random noise, theory-of-mind = stateless defaults, longitudinal tables = 0 rows). Landed on the honest on-device baseline instrument for Watch owners, built only on the verified real assets. All strategy lenses scored feasibility 3–5/10 — because the hardness is distribution + durable storage (founder-only), not code.
- **The cold-start unlock** — HealthKit historical backfill crosses the ≥10-sample gate from real retrospective data on grant, making "baseline from day 1" honestly true instead of fabricated.
