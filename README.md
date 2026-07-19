# Honest engineering skills for Claude Code

Nine skills that make an AI coding agent check its work instead of telling you what you
want to hear.

Every one exists because something went wrong. A model that looked like it worked was at
the chance ceiling once the splits were honest. An "implemented" engine turned out to be
`Math.random()` behind a confident interface. A test gate reported green because its exit
code was swallowed by a pipe. These are the procedures that came out of those.

## The skills

| Skill | Use it when |
|---|---|
| [honest-negatives-validation](skills/honest-negatives-validation/SKILL.md) | Before believing any accuracy, F1 or "the model works" claim |
| [stub-detection-audit](skills/stub-detection-audit/SKILL.md) | Before calling anything "done", "real", or "implemented" |
| [ml-feedback-provenance](skills/ml-feedback-provenance/SKILL.md) | Anything that feeds predictions back into training |
| [ship-gate](skills/ship-gate/SKILL.md) | Before committing or merging non-trivial code |
| [security-triage](skills/security-triage/SKILL.md) | Triaging a large set of scanner or audit findings |
| [product-honesty](skills/product-honesty/SKILL.md) | Deciding whether a claim is buildable on the real substrate |
| [deploy-target-discipline](skills/deploy-target-discipline/SKILL.md) | Before any deploy, or when production "looks reverted" |
| [vitals-signal-validation](skills/vitals-signal-validation/SKILL.md) | Camera or wearable vital-sign extraction (rPPG, HRV, SpO2) |
| [fable-economy](skills/fable-economy/SKILL.md) | Multi-change build work — walk the reuse ladder first |

## The idea

Coding agents fail in a specific direction: they report success. Tests "pass" because the
assertion was weakened. A feature is "implemented" because the file exists and exports the
right names. Accuracy is "good" because nobody wrote down the majority-class baseline
first.

None of that is lying, exactly — it's the absence of a procedure that would have caught
it. These skills are those procedures, written as checklists an agent has to actually walk.

Three that carry most of the weight:

**honest-negatives-validation** — pre-register the split, the baseline and the success
threshold *before* looking at the number. A metric chosen after seeing the result isn't
evidence. If accuracy sits at or below the majority-class baseline, that's a negative, and
you report it as plainly as you'd report a win.

**stub-detection-audit** — read the leaf methods, not the structure. A module can have the
right architecture, the right names, and full test coverage while its core returns a
random number. Structure tells you nothing about whether anything computes.

**deploy-target-discipline** — production "reverting to an old version" is almost never
git losing your code. It's a second deployment target you forgot about, or a build that
silently failed. Find the one canonical target and verify with a route only the new code
has.

## Install

Copy the ones you want into your skills directory:

```bash
git clone https://github.com/QEbellavita/claude-honest-engineering-skills
cp -r claude-honest-engineering-skills/skills/* ~/.claude/skills/
```

Or take a single one:

```bash
cp -r claude-honest-engineering-skills/skills/ship-gate ~/.claude/skills/
```

Each is a self-contained `SKILL.md` with YAML frontmatter — no dependencies, nothing to
build. They cross-reference each other with `[[wiki-links]]`; those resolve if you install
the whole set and degrade harmlessly if you don't.

## Adapting them

The `## Real cases` sections at the end of several skills describe failures from my own
work, genericised. They're there because a checklist without a war story behind it reads
as bureaucracy — you follow a rule you understand the cost of breaking.

Replace them with your own as you accumulate them. That's the intended use: these encode
one engineer's scar tissue, and yours will differ.

## Licence

MIT — see [LICENSE](LICENSE).
