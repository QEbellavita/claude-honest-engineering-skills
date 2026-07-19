---
name: vitals-signal-validation
description: Validation discipline for camera/wearable vital-sign extraction (rPPG/iPPG heart-rate, HRV, SpO2) BEFORE you trust an HR number or ship an extractor. Use when building or evaluating a signal extractor, running POS/CHROM/GREEN or a deep-rPPG model, or judging "the HR looks right". Catches the traps that produce confident-wrong vitals: dead chrominance, ROI locked on background, codec/harmonic frequency locks, and n-of-1 false confidence.
---

# Vitals-Signal-Validation

A heart-rate number from a camera is guilty until proven innocent. The failure mode is not noise — it's a **confident wrong number**: the FFT peaks cleanly on a codec artifact, a background flicker, or a harmonic, and the extractor reports 120 bpm with high confidence when the truth is 68. This skill is the pre-flight that separates a real pulse from a plausible artifact. It's the signal-processing sibling of [[honest-negatives-validation]] (which validates the *accuracy metric*); this validates the *signal* before any metric is computed.

**Keywords**: rPPG, iPPG, camera HR, heart rate, HRV, SpO2, POS, CHROM, GREEN, PhysNet, BVP, waveform, SNR, confidence gate, face ROI, demosaic, ground truth, ppg_sync.

## The Rule

**Trace the reported HR back to the frequency peak that produced it, and prove that peak is a pulse — not a codec, a harmonic, or the background.** A clean-looking spectrum is not evidence; a peak at exactly `fps/N · 60` or at 2× the true rate is the default suspect.

## Checklist (create one todo per item)

1. **Is there chrominance at all?** Chrominance methods (POS/CHROM) are mathematically dead on grayscale input. If frames are `R=G=B` (e.g. Rice `png/` are grayscale) the color lives only in the raw Bayer (`*.pgm`) — you MUST demosaic (`cv2.COLOR_BAYER_BG2BGR`) first. A "POS failure" is often just a color-blind input. GREEN dying while POS survives is the tell that lighting cancellation matters; GREEN dying *with* POS is the tell that there's no color.

2. **Is the ROI actually on skin?** Haar `detect_face` (largest-box) locks onto the **background** on downward-gaze or off-center scenes, and reports HR from wall flicker. Verify the box lands on skin every clip. Prefer a YCrCb **skin-segmentation** ROI (`Cr∈[133,180], Cb∈[77,130]`) that auto-tracks over largest-box Haar. If Haar is used, re-detect periodically with a last-good-box fallback, and log the fallback rate — a whole-frame fallback is a whole-frame spatial mean, which is garbage on a full scene.

3. **Whole-frame vs cropped-patch mismatch.** An extractor tuned on pre-cropped skin patches (rPPG-10 64×64 forehead) will produce ~38 bpm MAE on full 640×480 webcam frames because it spatial-means the whole scene. Confirm the input granularity the extractor *assumes* matches what you're feeding it. (This exact mismatch turned MAE 37.9 → 3.9 once a detect-and-crop ROI mode was added.)

4. **Check for frequency locks before believing any peak.**
   - **Codec spike**: a peak at exactly `fps/12 · 60` (e.g. 24fps → 120.1 bpm) is an mpeg4 GOP=12 compression artifact winning the FFT, not tachycardia. Do NOT "fix" it with a blanket notch filter — that regresses genuine high-HR subjects. The real lever is a better ROI; and a good confidence gate already abstains on these (they show lowest SNR / lowest confidence / widest cross-method spread).
   - **Harmonic doubling/halving**: the contact-PPG *reference* itself can double (2nd harmonic rivals the fundamental) — so a "sub-harmonic capture" bug can live in the ground truth, not the extractor. Use a robust FFT that reports `f_top/2` when that sub-multiple has ≥0.5× the peak power, and verify against a periodogram, not plain argmax.

5. **Score against the right ground truth, the right way.** Trust the **time-aligned windowed MAE**, never whole-clip FFT HR — even the GT's own whole-clip FFT hits harmonics (Rice: GT FFT 75.0 vs GT windowed-median 77.2). Confirm *which* reference column is frame-synced (mcd `ppg_sync` = col 0; the clinical `pulse`/`pulse_gt` scalar is a spot reading at a different time and will NOT correlate — that's expected, not a bug). Exclude dead/implausible references (flat-line sensor, corrupt sync) with a `valid_reference` guard before computing aggregate error.

6. **SNR/confidence is the deployment lever, not an estimator patch.** SNR predicts accuracy near-perfectly (SNR>0 → sub-bpm MAE; SNR<−5 → MAE ~6+). The honest move is to **gate** (abstain below a confidence threshold) rather than patch the estimator to force a number. Report both un-gated and gated MAE with the kept-fraction (e.g. "MAE 1.77 ungated → 0.44 at conf≥0.5, 95/160 kept").

7. **Guard your confidence in the result itself.** n=1 subject with 4 conditions is a *spot-check*, not validation — across-subject spread is the real open gap. If dataset licensing is unconfirmed, it's **eval-only, ship nothing**. State the n and the license status next to every number.

## Reporting

- Give HR with its provenance: method, ROI mode, SNR/confidence, windowed-MAE vs which reference, n subjects. Not just "77 bpm."
- Name any artifact you ruled out ("the 120.1 peak is the fps/12 codec spike, gated out at conf 0.30 — not tachycardia").
- Separate "extractor is wrong" from "reference is wrong" from "input is degenerate" — three different fixes.
- Falsify the tempting single-lever fix before adopting it (tighter crop, blanket notch, windowed-median all *regressed* clean clips here — adversarial-verify caught each).

## Real cases

- **rPPG-10 / mcd** — whole-frame spatial mean → MAE 37.9; detect-and-crop ROI → 3.9; scaled n=160 → 1.77 ungated / 0.44 gated. The two hard fails were the fps/12 codec spike, already abstained on by the confidence gate. Fine-tuned PhysNet (in-domain, codec artifacts present) beat POS on weak clips (18.9→3.0) — off-the-shelf PhysNet did NOT transfer (DiffNormalized amplifies the codec artifact).
- **A single-subject iPPG bakeoff** — grayscale `png/` = dead chrominance (demosaic the Bayer `pgm`); Haar locked on the downward-gaze background (YCrCb skin-seg fixed it, 0 fallback); windowed-MAE 0.92 median, motion penalty ~5× but no half-rate lock. eval-only (license unconfirmed), still n=1.
