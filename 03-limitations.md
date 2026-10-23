# Limitations and scope of claims

**Read this before reading any number in this package.** Every metric here is a
**prediction**, produced by a model, on designs that have not yet been made or measured.
The purpose of this section is to state precisely what each number can and cannot support,
including where our own evidence is weak.

---

## 1. Weak-proxy register

Each metric below is a **relative ranking device within one target and one pipeline**. None
of them is a calibrated predictor of binding, and none has been validated against wet-lab
data for these designs.

| metric | what it can support | what it **cannot** support |
|---|---|---|
| **pDockQ2** | ordering candidates *within* this pipeline and target | an absolute affinity; cross-target thresholds; a probability that a design binds |
| **ΔΔE (pH class)** | which side of a switch a design falls on, relative to the calibration anchors | a quantitative pH-dependent affinity; a guarantee the switch will appear in a real assay |
| **agonist-footprint Jaccard (C3)** | contact overlap with the native agonist footprint — used as **negative** evidence | whether a design **activates** the receptor. Contact overlap is not activation; a design can overlap the footprint and be an antagonist, and a design can activate through non-overlapping contacts |
| **ipTM / ipSAE** | secondary interface sanity checks | ranking, since `max pDockQ2` outperformed them on the 16-case set |
| **self-consistency (CA-RMSD)** | nothing predictive here — see §4 | any ranking use |
| **developability screen** | flagging sequences with motif-level risk | expression yield, solubility, or aggregation in a real host |

## 2. Statistical power is weak, and we state it rather than smooth it

- **The 16-case evaluation set has 3 positives.** AUROC 0.9744 on n=16 is coarse: a single
  rank swap moves it by ≈0.026. This is a **ranking sanity check, not a significance test**.
- **The key was the top of 39 candidate keys tried.** Selection bias is therefore present in
  the 0.9744 figure and must not be read as out-of-sample performance.
- **No threshold was fitted**, and we do not claim one. Every use of pDockQ2 in this package
  is ordinal.
- **The cross-target pH result is asymmetric in strength** (see §3). One target is
  significant; the other is not, and we say so.

## 3. C2's pH correlation is under-powered — do not read it as confirmed

In the cross-target comparison (`02-key-results.md` §4):

- C1: Spearman |ΔΔE| vs pDockQ2 = **−0.387, p ≈ 0.049**
- C2: Spearman |ΔΔE| vs pDockQ2 = **−0.049, p ≈ 0.79**, with **only 2 BASE designs** in the
  comparable set.

**n=2 makes a rank correlation nearly powerless**, so C2's non-significance should be read
as *absence of evidence*, not evidence of absence. The cross-target claim is carried by the
**count-based observation** that both challenges put 7 of their top 8 designs in the
NEUTRAL class — a fact that does not depend on C2's p-value. Presenting C2 as a second
statistically significant confirmation would be a misstatement, and we do not.

## 4. Self-consistency carries no signal here

Mean pairwise CA-RMSD between diffusion samples shows **no significant association** with
pDockQ2 on any target (rho −0.06 to −0.25, all p > 0.2). The samples of one design often
adopt **different binding modes** (means 6–33 Å). Two consequences:

1. it is **not** used as a ranking key;
2. the predictions should be understood as *a* plausible binding mode, not *the* binding mode.

## 5. Backbone count does not buy footprint overlap

More RFD3 backbones raise the **maximum** score but leave the **median** Jaccard essentially
unchanged, because RFD3 cannot constrain contact patterns. We report this as a negative
result and make **no claim** that a larger backbone pool improves agonism prospects. Where
backbone count and hotspot definition moved together in our batches, the two effects cannot
be separated, so no length/definition interaction is claimed either.

*(One un-controlled observation, reported as an observation only: designs built on
90-residue backbones show higher Jaccard than their 70-residue counterparts — e.g. core8_90
median 0.3605 vs core8_70 0.1812, and all six frozen designs with Jaccard ≥0.5 use 90-residue
backbones. This was not a controlled experiment and is not presented as a design rule.)*

## 6. The same-target calibration is fair but not absolute

The cetuximab Fab control (pDockQ2 0.4354) was run under identical conditions, so the
comparison is like-for-like. But:

- **Antibodies are systematically under-predicted in single-sequence mode**, because CDR
  conformation depends on somatic hypermutation and evolutionary information. 0.4354 is
  therefore **not** cetuximab's real binding capability — it is a same-conditions reference.
- **pDockQ2 is geometry-dependent and not fully size-invariant.** A 422 aa Fab and a
  70–110 aa designed binder present different interface geometry, which affects the score
  independently of binding quality.
- The most important limitation: **"high pDockQ2" and "actually binds" are both predictions.**
  The calibration establishes a relative scale, not an absolute guarantee.

## 7. C3 agonism is the least supported claim in the submission

C3 awards the majority of its points for **receptor activation**, and **no predictor for
activation was available to us**. The footprint Jaccard is a weak proxy used as *negative*
evidence only. We mitigate this by splitting the 24 slots into three classes (A strong-score/
strong-footprint, B weaker-score, C strong-score/weak-footprint) precisely so that the
wet-lab outcome can be attributed to a specific step — class C is the arm that tests whether
the footprint proxy is wrong. **This is a hedge against a known unknown, not a solution to it.**

## 8. What the developability screen does not cover

The screen is sequence-level and motif-level. It does **not** predict:

- expression yield or solubility in any particular host;
- aggregation propensity under real assay conditions (the GRAVY and low-complexity numbers
  are proxies, not measurements);
- immunogenicity, stability, or proteolytic sensitivity;
- whether a predicted fold is realised in vitro.

No design in this submission has been synthesised, expressed, or assayed.

## 9. Provenance and reproducibility limits

- The C2 batch-2 Boltz-2 predictions were **split across two GPUs** to cut wall-clock time.
  Scoring settings were identical (single-sequence, recycling 3, sampling 200,
  `--no_kernels`, 3 diffusion samples); only the random seed differed per segment
  (20261035/20261036/20261037). The frozen ranking key is the max over a design's own three
  samples, so it is seed-independent — but the split is recorded rather than left implicit.
- Designs were replaced **once**, after the developability gate found poly-Ala runs. The
  substitution is recorded per row (`replaced`, `replaced_reason`) in the lineage tables.
  Any analysis of this submission must account for those replacements.
- Intermediate artifacts (all predicted structures, per-sample metrics, pool tables) exist
  on the compute host; this package ships the submitted designs' selected structures, the
  targets, the reference control, and the metrics tables.

## 10. Scope statement

These designs are de novo proteins produced by a generative pipeline and scored by
structure-prediction models. They are **hypotheses for experimental testing**. The
submission makes **no claim of demonstrated binding, activity, or selectivity**, and no claim
that the predicted poses are the true poses. The value of the set is that its designs are
traceable: every choice — hotspot definition, backbone, sequence, pH class, hedge class —
can be traced from a wet-lab readout back to the step that produced it, which is what makes
the result interpretable whichever way the assay goes.
