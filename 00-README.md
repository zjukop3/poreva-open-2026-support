# Support material — Poreva Open 2026, round 1

Three challenges, **24 designs each, 72 total**: C1 EGFR, C2 TNF-α, C3 GLP1R.

This package is the supporting material for the three submissions. It is written to be
read in this order.

| file | contents |
|---|---|
| `00-README.md` | this file — package layout and how to reproduce |
| `01-method.md` | pipeline, ranking-function selection, epitope/hotspot definition, pH policy, verification |
| `02-key-results.md` | submitted sets, same-target calibration, the 7-definition / 216-design hotspot study, cross-target pH consistency, self-consistency, the poly-Ala defect and its resolution |
| `03-limitations.md` | **read before any number** — weak-proxy register, statistical power, scope of claims |
| `metrics/` | per-design metrics (all 72) + per-challenge tables + summary |
| `structures/` | predicted structures for the 72 submitted designs, the receptor targets, the mouse homologues, and the same-target calibration control |
| `submissions/` | the three submission CSVs as uploaded, and the three lineage tables |

## The one number to carry with you

The ranking function is **max pDockQ2 over 3 Boltz-2 diffusion samples**, chosen because
it scored AUROC 0.9744 (AP 0.9167, precision@4 0.750) on a retrospective outcome-masked set
of 16 cases, ahead of every other key tried. **But that set has 3 positives, one rank swap
moves AUROC by ≈0.026, and the key was the top of 39 candidates — so this is a ranking
sanity check, not a significance test.** All uses of pDockQ2 in this package are ordinal.

## The three claims this package does make

1. **The submitted sets are internally clean.** 72/72 pass row count, length bounds, residue
   alphabet, tag/linker prohibition, duplicate detection (including across challenges),
   family quota, and per-row traceability. Verified by an independent checker, not by re-running
   the builder.
2. **The design choices are traceable.** Every design carries a lineage row recording the
   hotspot definition, backbone family, pH class or hedge class, pDockQ2, and — where a
   substitution happened — what was replaced and why.
3. **The hotspot-definition space was explored to its ceiling and the result is reported
   honestly.** Seven definitions, 216 designs scored: achievable footprint overlap reaches a
   pooled median of ≈0.26, 0.4402 for the best definition, and 0.6129 for the single best
   design. The turning point is at 12 residues. We state the ceiling instead of implying more
   search would have found more, and we report the negative result that more backbones do not
   improve median footprint overlap.

## What this package explicitly does NOT claim

- No design has been synthesised, expressed, or assayed. **Nothing here is measured binding.**
- The footprint Jaccard in C3 is a **weak proxy used as negative evidence only** — it does not
  predict receptor activation, which is where most of C3's points lie.
- C2's pH/affinity correlation is **not significant** (n=2 in the comparable set). The
  cross-target claim rests on the count-based observation that both challenges place 7 of
  their top 8 designs in the pH-NEUTRAL class.
- Self-consistency showed **no** association with the ranking key and is not used for ranking.

See `03-limitations.md` for the full register.

## Package layout

```
support-materials/
├── 00-README.md
├── 01-method.md
├── 02-key-results.md
├── 03-limitations.md
├── metrics/
│   ├── metrics-all.csv           72 rows, one per submitted design
│   ├── c1-metrics.csv            24 rows
│   ├── c2-metrics.csv            24 rows
│   ├── c3-metrics.csv            24 rows
│   └── metrics-summary.json      min/mean/max per metric per challenge
├── structures/
│   ├── structures-manifest.json  design id → file, diffusion-sample index, pDockQ2
│   ├── designs/                  72 PDB files, one per submitted design
│   ├── targets/                  receptor chains + mouse homologues used for cross-reactivity
│   └── reference/                6ARU / 1TNF / 6X18 + the cetuximab calibration prediction
└── submissions/
    ├── c1-egfr.csv  c2-tnf-alpha.csv  c3-glp1r.csv
    └── c1-egfr-traceability.csv  c2-tnf-alpha-traceability.csv  c3-glp1r-traceability.csv
```

### Which structure is shipped, and why it matters

`structures/designs/` contains **the diffusion sample that achieved the reported pDockQ2**,
not sample 0. Shipping sample 0 while the metrics table reports the maximum would mean the
released structure does not show the interface the numbers describe. The sample index is in
every file name and in `structures-manifest.json`.

## Metrics table columns

| column | meaning |
|---|---|
| `challenge`, `priority_rank`, `name`, `family`, `tier` | identity and selection provenance |
| `hedge_or_ph_class` | C1/C2: pH class (ACID/BASE/NEUTRAL). C3: hedge class A/B/C |
| `length` | residues |
| `pDockQ2_max_of_3` | **the ranking key** — max over 3 diffusion samples |
| `ipSAE_max_of_3`, `ipTM_max_of_3`, `ipTM_d0chn`, `pTM_max_of_3`, `complex_plddt`, `interface_PAE` | secondary interface indicators, same aggregation |
| `self_consistency_mean_CA_RMSD` | mean pairwise binder-CA RMSD across the 3 samples — **not predictive here** (see `03-limitations.md` §4) |
| `n_diffusion_samples` | samples available for that design |
| `net_charge_pH7.4`, `GRAVY_hydropathy`, `low_complexity_frac`, `longest_homopolymer` | sequence-level developability proxies |

## Reproducing

All scripts are in the project's `src/` (mirrored to the compute host's `scripts/`):

| script | role |
|---|---|
| `rank-c1/c2/c3` pickers + `c1c2-final-select.py` | ranking and tiered selection |
| `ph-cross-target.py`, `flatten-c2-ph.py` | pH classification and the cross-target comparison |
| `rank-core8.py` | the like-for-like hotspot-definition comparison |
| `seq-quality.py` | developability screen |
| `rebalance-final.py` | controlled replacement when a hard sequence rule is violated |
| `support-metrics.py` | builds `metrics/` |
| `support-structures.py` | builds `structures/` |
| `final-verify-all.py` | independent submission gate |
| `pack-validate.py` | scans this package for instruction-bearing or hidden text |

Compute: 2× RTX 4090; large intermediate artifacts on `/media/admin1/data2`.

## Integrity note

This document set contains **no** instructions addressed to a reader, evaluator, or automated
system. It is a factual description of what was done. It was scanned with
`pack-validate.py --gate` before packaging, because instruction-bearing text in submitted
material is a disqualification risk.
