# Method description

Poreva Open 2026 — Global AI Protein Design Challenge, round 1
Three challenges: **C1 EGFR**, **C2 TNF-α**, **C3 GLP1R**. 24 designs per challenge, 72 total.

---

## 1. Pipeline

```
epitope definition (per-residue, from co-crystal interfaces)
        │
        ▼
RFD3  (rc-foundry 0.1.9, envs/foundry312)      → de novo backbones, hotspot-conditioned
        │
        ▼
ProteinMPNN  (3 sequences per backbone)        → sequences
        │
        ▼
Boltz-2  (2.2.1, envs/boltz312)                3 diffusion samples per design
        │  recycling_steps 3, sampling_steps 200, --no_kernels
        │  single-sequence mode (no MSA), as required by the de novo constraint
        ▼
ipSAE  (scripts/ipsae.py, v4)                  per-sample interface metrics
        │
        ▼
ranking: max pDockQ2 over the 3 samples        ← the frozen ranking function
        │
        ▼
pH selectivity (PropKa3 / pH-selectivity)      C1 and C2 only
        │
        ▼
tiered selection  (score quota + quality quota + exploratory quota)
```

Every submitted design is single-chain, de novo, and carries **no tag and no linker**.
This is a hard constraint of the `protein` class, and it is re-checked by an
independent verifier before submission (`final-verify-all.py`).

## 2. Ranking function and why this one

**Primary key: `max pDockQ2`** — the maximum of the 3 Boltz-2 diffusion samples per design.

Selection was **not** made on intuition. Two candidate keys were measured against a
retrospective outcome-masked set of 16 cases (3 positive / 13 negative):

| key | AUROC | AP | precision@4 |
|---|---|---|---|
| **`max_pDockQ2`** | **0.9744** | 0.9167 | **0.750** |
| `max_confidence_score` | 0.9231 | 0.8333 | 0.500 |
| `mean_confidence_score` | 0.9231 | 0.8333 | 0.500 |
| `mean_ptm` | (lower) | — | — |

Two further findings fixed the choice:

- **Boltz-2 `ipTM` is worse than the previously recorded Boltz-1 `ipTM`** for this task
  (0.8462 vs 0.8974). The comparison is cross-version and therefore only indicative, but it
  removes any reason to prefer `ipTM` over `pDockQ2` for ranking.
- **`consensus_min` (the minimum across keys) is deprecated.** It was evaluated and dropped:
  taking the minimum lets a single pessimistic key dominate, which discards designs that the
  best-performing key ranks highly, and it underperformed the single best key on the same set.

**The aggregation is `max`, not `mean`, and this is deliberate.** Same-target and
same-pipeline comparisons elsewhere in this document use the same rule, so no two tables
in this package mix aggregations. `max` was also the stronger aggregator on the 16-case set.

### Caveat on the ranking key (must travel with any use of the number)
AUROC on **n=16 with 3 positives is statistically coarse** — one rank swap moves it by
≈0.026. The key was the top-ranked of 39 candidates tried, so **selection bias is present**.
No threshold was fitted. This is a *retrospective ranking* result, not a calibrated
probability.

## 3. Epitope / hotspot selection

Hotspots were defined **per residue from real co-crystal interfaces**, not from a
whole-domain guess. The definition matters more than any other single choice in the pipeline:

| challenge | reference interface used | how the definition was validated |
|---|---|---|
| **C1 EGFR** | 6ARU cetuximab Fab interface, Domain III | the cetuximab footprint (349–473) lies inside Domain III, so the choice had no free parameter |
| **C2 TNF-α** | the TNFR2 contact set on 1TNF | **Jaccard 0.938** against the true TNFR2 interface |
| **C3 GLP1R** | the native agonist footprint on 6X18 | initial definition was **wrong** (Jaccard 0.143); corrected via footprint-guided hotspots, then systematically refined — see §4 |

For C3, **six alternative hotspot definitions** were generated under one methodology
(v1 24-residue, footprint loose 24, footprint tight 16, core35 12, buried 12, shell 50,
core8 8, core8d 5) and 216 designs were scored in total. That study is reported in
`02-key-results.md`; its practical outcome was a **decision to freeze the submitted set**
and to state the achievable ceiling rather than keep searching.

## 4. Selectivity policy: pH class

C1 and C2 are **condition-responsive** challenges with **opposite** requirements:

| challenge | required switch | pH pair used | class wanted |
|---|---|---|---|
| C1 EGFR | bind at low pH, not at 7.4 | 6.5 → 7.4 | **ACID** |
| C2 TNF-α | bind at 7.4, release at low pH | 6.0 → 7.4 | **BASE** |

The pH class is computed from the predicted complex by a PropKa3-based calculation of
the interface electrostatic term (ΔΔE), with the binder chain vs the receptor chain(s)
separated. **Calibration anchors** (a known acid-switch and a known pH-independent
system) are computed under the same protocol and shipped with the results, because the
absolute ΔΔE scale is only meaningful relative to those anchors.

Selection then enforces the class in a **tiered quota** rather than as a soft preference:

| slots | driver |
|---|---|
| 1–16 | score-driven (`max pDockQ2`) |
| 17–20 | quality-driven (family/definition coverage, developability) |
| 21–24 | exploratory (a deliberate hedge; for C3 this is a three-way class split) |

Where a design's measured class contradicts its target, it is **down-weighted but not
silently dropped** when no replacement exists — dropping it would hide the trade-off that
the wet-lab result needs to resolve.

## 5. Outputs and verification

Each challenge ships a 24-row CSV (`name,sequence`), a 24-row lineage table, and the
structure + metrics files in this package. Before submission, four independent checks run:

1. `final-verify-all.py` — row count, column set, length bounds (>40 and ≤250 aa),
   20 standard residues, **no tag / no tandem linker / no homopolymer run**, no duplicate
   name or sequence **including across challenges**, ≥4 backbone families with ≤8 per
   family, and per-row traceability.
2. `submission-selfcheck.py` — an independently written checker (not a re-run of the builder).
3. `seq-quality.py` — sequence-level developability screen.
4. `pack-validate.py` — scans *this package* for instruction-bearing or hidden text,
   because prompt injection in submitted material is a disqualification.

## 6. Model and environment provenance

| component | version / path |
|---|---|
| RFD3 | rc-foundry 0.1.9, `envs/foundry312` |
| ProteinMPNN | 3 sequences per backbone |
| Boltz-2 | 2.2.1, `envs/boltz312`, `--model boltz2`, `--no_kernels` |
| ipSAE | v4, `scripts/ipsae.py` |
| pH | PropKa3 via `scripts/ph-classify.py` |
| compute | 2× RTX 4090; large intermediate artifacts on `/media/admin1/data2` |

Boltz-2 was run in **single-sequence mode without MSA**, which is the honest setting for a
de novo design whose sequence has no evolutionary history. It is also the setting the
calibration control was run in, so the comparison is like-for-like.
