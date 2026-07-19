---
name: ship-gate
description: The pre-merge quality gate for any feature or bugfix. Use when completing an implementation, before committing nontrivial code, or before merging/opening a PR. Enforces TDD-first, then an adversarial review of the finished diff, then fixing findings, then merge. Catches happy-path lies, auth holes, and tenant-isolation bugs before they ship.
---

# Ship-Gate

Working code is not shippable code. The gate exists because adversarial review of *finished* work catches what the author cannot see — a C1 auth bug reached "done" and was only caught at this gate before ship. Nothing merges until it passes.

**Keywords**: merge, PR, ship, done, complete, review, gate, pre-merge, TDD, deep-review, ready to commit.

## The Sequence (do not reorder)

```
TDD (test first) → implement → adversarial review of the diff → fix findings → re-verify → merge
```

## Checklist (create one todo per item)

1. **Test-first.** Before implementation, write the failing test that defines "done." If you're bugfixing, write the test that reproduces the bug and fails. Never write the implementation before the test — the test is the spec.

2. **Implement to green.** Make the test pass. Run the *full* suite, not just the new test — record before/after counts (e.g. 277 → 309). A drop in unrelated tests is a regression, not noise.

   **Gate integrity — never mask the exit code.** A gate you can't trust is worse than no gate. Do NOT pipe a test/gate command through `tail`/`grep`/`head` — without `pipefail` the pipeline's exit code is the *filter's*, not the runner's, so a failed suite reports exit 0 (this nearly shipped a false green on #327: `test:ci | tail -6` returned 0 while 13 specs failed, and the pipe also truncated the log to uselessness). Instead: write full output to a file (`> log 2>&1`), echo `exit:$?` explicitly, and read counts from the file. When a gate fails right after a merge, immediately run the *same command on a pre-change control tree* to classify regression vs pre-existing drift before you act on it.

3. **Adversarial review of the finished diff.** Hand the diff to a fresh, skeptical reviewer (deep-reviewer) whose job is to *break* it, not bless it. The review MUST attack, at minimum:
   - **Auth** — every new route: is it actually guarded? Can it be hit unauthenticated or with a stale/forged token?
   - **Tenant isolation / BOLA** — can user A read or mutate user B's data by changing an ID? Is the query scoped to the caller?
   - **The happy-path lie** — does it only work on the golden input? What happens on empty, malformed, concurrent, or hostile input?
   - **Does the test actually test it** — or does it assert on a mock / fabricated success? (See [[stub-detection-audit]].)
   - **Claimed vs actual** — does the code do what the PR says, or does it no-op and report success?

4. **Fix every confirmed finding before merge.** A finding is not "noted for later" — it's a blocker. Re-run the reviewer or re-verify after the fix. Only genuinely-out-of-scope items get deferred, and only with a written note.

5. **Re-verify end-to-end.** Drive the actual flow (not just tests) — see the change work. Then confirm the suite is green and counts are right.

6. **Merge.** Only after 1–5. On a security-sensitive area, the gate is mandatory, not optional.

## When to invoke a heavier gate

- **Security-sensitive diff** (auth, payments, user data, tenant boundaries) → full adversarial review is required, and apply [[security-triage]] to any finding set. Where money or regulated data is involved, hold a bank-grade bar.
- **ML/signal change** → also run [[honest-negatives-validation]] on any accuracy claim in the diff.
- **"Is this feature even real"** doubt → run [[stub-detection-audit]] first.

## Real cases

- **C1 auth bug** — reached "done," caught by the adversarial gate pre-ship. The reason the gate is non-optional on auth.
- **PRs #321–322** (provenance guard, feedback sweep) — TDD'd, deep-reviewed, findings fixed, then merged; suite 889 → 798+91.
- **PR #324** (P0 security) — 150/150 green, adversarial review surfaced 3 broken paths (watch/sync auto-enroll, emotion-memory softAuth, social-contagion) that were fixed before merge.
