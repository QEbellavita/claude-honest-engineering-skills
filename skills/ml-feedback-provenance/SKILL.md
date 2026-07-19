---
name: ml-feedback-provenance
description: Discipline for guarding a self-learning ML loop — outcome feedback, weight recompute, shadow-mode promotion, and ground-truth sourcing — so models don't learn from their own guesses or promote themselves on rigged evidence. Use when designing, reviewing, or changing anything that feeds prediction outcomes back into model weights, promotes a shadow model, scores accuracy, or retrains. Catches feedback famine, the resurrection hole, pseudo-label poisoning, and self-agreement promotion.
---

# ML-Feedback-Provenance

A learning loop that trusts the wrong signal doesn't fail loudly — it quietly optimizes toward its own bias. The dangerous cases here all *looked* like a working flywheel: models weighted by "feedback" that was actually their own pseudo-labels; a shadow model promoted because it *agreed with production*; a discredited model creeping back to nonzero weight. This skill is the provenance discipline that keeps the loop honest. It's the ML-ops sibling of [[security-triage]] (a specific failure class with a specific checklist) and pairs with [[honest-negatives-validation]] (is the accuracy number real) and [[stub-detection-audit]] (is the model even real).

**Keywords**: feedback loop, outcome feedback, weight recompute, ensemble weights, shadow mode, model promotion, ground truth, pseudo-label, self-evolution, retrain, provenance, calibration, discredited model, F1 gate.

## The Rule

**Every weight change and every promotion must trace to a ground-truth signal that is independent of the model being scored.** If a model's weight can rise because of its own output — directly (pseudo-labels), transitively (another model's guess), or by agreement with production — the loop is rigged. Sever that path before trusting anything downstream of it.

## Checklist (create one todo per item)

1. **Audit the ground-truth allowlist.** What counts as "truth" for accuracy scoring? Only genuine signals qualify — an explicit self-report is truth; a temporal-oscillation sentinel, a behavioral-contradiction inference, or another model's pseudo-label (`amigos_ensemble`) is NOT. Pseudo-labels can't equal a real emotion and they bias weights toward the floor. Enforce an explicit `GROUND_TRUTH_SOURCES` allowlist; anything not on it does not move weights.

2. **Kill self-agreement promotion.** A shadow/candidate model must NOT be promoted because it agrees with the current production model — that just clones the incumbent's bias. Promotion gates on a **labeled holdout** eval (min accuracy, baseline-delta, coverage), not prod-vs-candidate agreement. A "shadow safety" check is an operational veto only — it never reads production-vs-candidate agreement as evidence of quality. (This is the Option-C design shipped in #329.)

3. **Fail closed on unsupported or vacuous gates.** A promotion gate that can't be evaluated must **block**, not wave through. `min_f1_macro` on a metric that isn't computed = fail-closed `unsupported_gate`, not pass. `min_accuracy <= 0` is vacuous = fail. A gate that silently passes because it couldn't run is worse than no gate.

4. **Close the resurrection hole for discredited models.** A model marked discredited/banned must stay at weight 0 and be *enforced* there on every recompute — verify it can't drift back to nonzero through a recompute path that doesn't check provenance. (This was the #321 provenance-trust-guard fix: zero the discredited voters AND enforce the ban on recompute, not just once.)

5. **Check the recompute actually fires for everyone.** Count-gated recompute (e.g. `batchThreshold=50` paired predictions per user) starves low-volume users — their loop never closes and their weights never adapt. Confirm there's a path (a daily context-weight sweep, a global recompute) that reaches low-volume users, not just high-volume ones. Feedback famine is silent: the code runs, the learning doesn't.

6. **Watch for TOCTOU between eval and promotion.** The eval that passed and the criteria that gate promotion must be bound to the *same* artifact/commit — if a model can be re-evaluated or swapped between "eval passed" and "promote," the gate is bypassable. Bind eval↔criteria; #329 fixed exactly this residual after the first review.

7. **Don't wire the loop to a fabricated core.** Before feeding outcomes into an engine or trusting its forecast as a training signal, confirm the engine actually computes (run [[stub-detection-audit]]) — a feedback loop around a `Math.random()` forecast trains on noise. And know which prediction path you're on: one system I work on has two (JS in-process ONNX ensemble vs Python `.pkl`/`.joblib` API) that don't know about each other — don't assume a fix on one touches the other.

## Reporting

- State the provenance chain for any weight change: "weight rose because X; X is `explicit_self_report` (on the allowlist) — not a pseudo-label."
- For a promotion: the holdout eval numbers, which gates passed, and confirm no gate passed by failing-open.
- Name any model still dormant/discredited and that its weight is enforced at 0, not merely initialized there.
- Route the change through [[ship-gate]] with TDD — every one of these fixes shipped test-first and was deep-reviewed (some twice: DO-NOT-SHIP → fixed).

## Real cases

- **#321 provenance trust guard** — zeroed discredited voters and enforced the ban on recompute (closed the resurrection hole).
- **#322 daily context-weight sweep + ground-truth allowlist** — fixed the count-gated feedback famine (low-volume users never closed the loop) and gated scoring to genuine ground truth only.
- **#329 honest gated model-promotion (Option C)** — labeled-holdout eval gates promotion; shadow-safety is operational-veto-only and NEVER reads prod-vs-candidate agreement; ml-eval fail-closes on unsupported/vacuous gates; eval↔criteria TOCTOU bound. Deep-reviewed twice (DO-NOT-SHIP → fixed → residual TOCTOU → fixed).
- **The "model router" anti-pattern** — presented as performance-conditioned routing, actually static hardcoded weights with shadow slots at 0. No feedback conditioned the selection at all.
