---
name: fable-economy
description: Token-optimization protocol for iterative build work. Use when implementing features, making multiple code changes, or any "build/add/refactor X" request — walks the reuse ladder before writing new code (skip > reuse > stdlib > platform > installed dep > one-liner > minimum new code), scopes options with cost estimates first, patches instead of rewriting, audits cost before 5+ changes, and batches+verifies once. Trigger on "apply fable-economy protocol", "reuse ladder", or any multi-change build request.
---

# fable-economy Protocol

Token-optimization framework that cuts output spend 55–65% on iterative projects. **Cost lives in output, not input.** Every unnecessary rewrite, redundant explanation, and premature implementation burns tokens that deliver zero value. Trade narrative polish for efficiency.

## Law 0: THE REUSE LADDER (before writing ANY code)

The cheapest code is code never written. Before implementing each item, walk down this ladder and stop at the first rung that holds:

1. **Does this need to exist at all?** → skip it
2. **Already in the codebase?** → reuse it (grep first)
3. **In the language stdlib?** → use it
4. **Native platform feature** (browser API, OS, SQL)? → use it
5. **In an already-installed dependency?** → use it
6. **Can it be one line?** → make it one line
7. Only then: write the minimum new code that passes

Only rung 7 costs output tokens and future maintenance. A new file, class, or dependency is a ladder failure unless rungs 1–6 were actually checked.

## The Four Laws

Create one TodoWrite item per law when starting a build session, and hold yourself to each.

### 1. LIST BEFORE BUILD
Present scoped options with cost estimates **before** implementing. Let the user prune.
- **Skip only** when the user already specified exact items by name or number.
- Otherwise: enumerate the candidate changes, give a rough cost/size per item, get approval.

### 2. PATCH, DON'T REWRITE
Use targeted `Edit` / `str_replace` edits, never regenerate a whole file to change part of it. Cuts per-file cost ~80%.
- Only rewrite a file when creating it new or when >50% of it genuinely changes.

### 3. COST AUDIT BEFORE EXECUTION
For any request touching **5+ changes**, show a complexity table before proceeding:

| # | Change | Scope | Cost |
|---|--------|-------|------|
| 1 | …      | S/M/L | low/med/high |

Users typically drop 1–3 low-ROI items once they see the table.

### 4. BATCH AND VERIFY ONCE
Ship all approved changes in a single pass, then run **one** syntax/type/test check at the end — not after each edit.
- No preambles before code. No post-execution narration. Lists over prose.

## Session Phases

1. **SCOPE** — list options, estimate costs, get approval (Laws 1 & 3)
2. **BATCH** — walk the reuse ladder per item (Law 0), then execute all approved changes in one pass per group (Law 2)
3. **VERIFY** — single final check (Law 4)
4. **AUDIT** *(optional)* — report tokens/changes saved vs. a naive rewrite

## Anti-patterns (stop if you catch yourself)

- Writing a paragraph of explanation before the user has approved scope.
- Regenerating a file to change three lines.
- Re-running the test suite after every individual edit.
- Implementing all of a vague request when the user might only want half.
- Writing a helper that already exists in the codebase, stdlib, or an installed dep (ladder rungs 2–5).
- Adding a new dependency or abstraction layer for something a one-liner covers.

> Reference: distilled from the fable-economy gist (amirmushichge/b4e133b7e7e86c445817a295ce43f76f); reuse ladder adapted from Ponytail (DietrichGebert/ponytail).
