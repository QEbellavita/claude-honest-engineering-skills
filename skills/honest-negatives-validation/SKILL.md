---
name: honest-negatives-validation
description: Pre-registered protocol for validating any ML / signal / accuracy claim before you believe or report it. Use BEFORE trusting or writing down that a model "works", predicts an outcome, or hits an F1/accuracy number — especially for biometric, EEG, affect, cross-subject, or wearable-signal work. Catches leaky splits, corrupted labels, and chance-level results dressed up as success.
---

# Honest-Negatives Validation

An accuracy number is a **claim**, not a fact, until it survives this protocol. Most impressive-looking ML results in this ecosystem died here: EEG affect that read as working was at the chance ceiling once splits and labels were fixed. Your job is to try to *falsify* the claim before repeating it. If you can't falsify it, then and only then report it — and report negatives just as plainly as positives.

**Keywords**: model accuracy, F1, validation, cross-subject, subject-independent, EEG, affect, valence, arousal, biometric, chance, baseline, leakage, GroupKFold, honest negative.

## The Rule

**Pre-register the method before you look at the result.** Decide the split, the baseline, and the success threshold *first*, in writing. A metric chosen or interpreted after seeing the number is not evidence.

## Checklist (create one todo per item)

Run these in order. A single failure invalidates the claim — stop and report the negative.

1. **Baseline first.** What is the majority-class accuracy (or trivial predictor)? Write it down before anything else. An "acc 0.549" means nothing until you know majority is 0.567 — that's a *loss*, not a win. Any accuracy at or below majority = NEGATIVE, full stop.

2. **Leak-free split.** Are train and test truly independent?
   - Cross-subject claim → **GroupKFold by subject**. Never a random split (the same subject's samples leak across folds and inflate everything).
   - Time series → split by time, no future-into-past.
   - If the claim is "cross-subject *and* cross-stimulus," hold out **both** subjects and clips — a model can memorize the stimulus and look subject-invariant.

3. **Verify the labels against the canonical source.** Do not trust labels embedded in a downloaded file.
   - Check the label *distribution* against the paper (a valence mean of 3.47 when canonical is 5.4 = corrupt mirror; recover from the original ratings file).
   - Check column *order* — a swapped `[arousal, valence]` vs `[valence, arousal]` silently trains the wrong target. Confirm which column is which.

4. **Stimulus-locked ≠ trait/state decoding.** If accuracy comes from clips/stimuli present in training, you're decoding the *stimulus*, not the person's felt state. Published "it works" numbers are often stimulus-locked. The honest test holds out the stimulus.

5. **"More data" is a hypothesis, not a fix.** If the claim is "it'll work with a bigger N," check whether it's already been tested at scale. If pooling corpora, account for domain shift (different hardware/montage = a confound, not free samples). Don't recommend "download more data" as the lever unless a scaled test actually moved the metric.

6. **Pre-register and record the outcome.** Write the method, baseline, and threshold to a `HONEST_*` record *before* the final run. Commit it. Report the result — positive or negative — against that pre-registration.

## Reporting

- State the baseline next to every accuracy: "acc 0.53 vs majority 0.53 = chance."
- Call a negative a negative. "Subject-independent X is at the chance ceiling" is a finding, not a failure.
- Name the real levers instead of hand-waving: personalization/calibration, domain adaptation, or fusing the weak signal as a prior — not "more data."

## Real cases

- **EEG affect ceiling** — 5 leak-free negatives; subject-invariant felt affect at chance up to **N=123 (FACED)**. "More data" falsified as the lever. FACED's own published ~35% 9-class is *clip-category* (stimulus-locked), not felt affect.
- **DEAP-Kaggle corruption** — local mirror had embedded labels with valence mean 3.47 (canonical 5.4). Recover from `Metadata/participant_ratings.xls`.
- **AMIGOS swap** — `labels_selfassessment` is `[arousal, valence, …]` (col0 = arousal); it was swapped in 4 loaders, inflating a leaky 0.618 that became an honest ~0.53 (≈ chance) after GroupKFold + correct columns.
