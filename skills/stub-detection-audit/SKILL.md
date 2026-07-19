---
name: stub-detection-audit
description: Audit procedure to determine whether a feature, engine, or endpoint is genuinely implemented or a fabricated placeholder. Use BEFORE declaring anything "real", "working", "done", or "implemented", before reporting a capability to the user, or when verifying claims against code. Reads leaf methods, not structure — catches Math.random() cores, no-op returns, hardcoded "results", untrained matrices, and writes that fake success.
---

# Stub-Detection Audit

Impressive structure is not implementation. An engine can be 1,000 lines of correct-looking state-space scaffolding whose actual math is `return input`. **Reading the structure fooled the first audit of the digital twin** — its Kalman update was a no-op and its anomaly score was `Math.random()*2`, but the class shape read as "real." The only reliable test is to read the **leaf methods** — the functions that supposedly do the computation — and confirm they compute.

**Keywords**: stub, placeholder, fabricated, no-op, fake, mock, is this real, capability verification, twin, forecast, router, does this actually work.

## The Rule

**Trace every claimed output back to the line that produces it, and read that line.** If the output originates from a random number, a constant, an untrained parameter, or an unmodified input, the feature is a stub regardless of how much code surrounds it.

## Checklist (create one todo per item)

1. **Find the output, walk backward.** Pick the value the feature claims to produce (a forecast, score, prediction, decision). Follow it to its source. Don't stop at the orchestrator — go to the leaf that computes it.

2. **Scan the leaf methods for fabrication tells:**
   - `Math.random()` / noise where computation belongs → the output is noise with an invented confidence.
   - No-op returns — `return input`, `return {}`, `return []`, identity functions named like they transform (`updateState`, `computeGain`) → the pipeline does nothing.
   - Hardcoded constants where data-fitting is claimed — `avgValence = 0.5`, fixed weights labeled "learned" → not trained.
   - Random-init matrices/params **never trained** (`observationMatrix` init'd random, no fit call) → output is structured noise.
   - Comment tells: "placeholder", "utility method", "TODO", "for now", "stub", "mock" sitting in a code path that's supposedly live.

3. **Check writes actually persist and reads actually query.** A POST/mutation handler that returns `{success: true}` without writing (swallows the payload) fakes success. A "search" that's a keyword `includes()` over a list is not semantic retrieval. Confirm the DB call / index lookup exists.

4. **Distinguish real fitting from decoration.** Some methods in a stub engine *are* real (e.g. circadian/weekly fitting) while the headline capability is fake. Don't let one genuine method certify the whole engine. Audit each claimed capability independently.

5. **Watch for sibling/ghost copies.** Search hits may land in an older sibling repo, a `-slice` frontend stub, or a worktree — not the live system. Confirm the file you're reading is the one that actually runs. A frontend `*-slice.ts` nav stub is not the backend service of the same name.

6. **Verdict per capability.** For each, state one of: **REAL** (leaf computes from data), **SCAFFOLD-WITH-FABRICATED-CORE** (structure real, math placeholder), or **ABSENT** (no code path at all). Cite the file:line of the deciding leaf method.

## Reporting

- Give the verdict with the smoking-gun line: "twin forecast = noise — `computeAnomalyScore` is `Math.random()*2` at emotional-digital-twin.js:1049."
- If you certified something "real" in an earlier pass on structure alone, **correct the record** explicitly — that's how the twin verdict got fixed.
- Separate "wired but fake" from "not wired" — they need different fixes.

## Real cases

- **Digital twin** — 1,097 lines of state-space structure; Kalman update no-ops, `computeAnomalyScore = Math.random()*2`, `observationMatrix` random-init & never trained → forecasts are noise. Structure fooled the first audit; leaf-method read corrected it.
- **`disputeTrait`** — POST handler swallowed the write and fabricated a success response.
- **"Model router"** — presented as context/performance-conditioned routing; actually static hardcoded ensemble weights with shadow slots at 0.
- **Intervention "RAG"** — a keyword inverted index over regime + VA quadrant, not vector/semantic search (no sentence-transformers/faiss/pgvector anywhere).
