# Poreva Open 2026 — Support Materials

Supplementary material package for our Poreva Open 2026 submissions
(three design challenges: C1 EGFR, C2 TNF-alpha, C3 GLP1R).

**Start here**: [`00-README.md`](00-README.md)

| Path | Contents |
|---|---|
| `00-README.md` | Package structure, how to read, headline numbers, what we do **not** claim |
| `01-method.md` | Pipeline (RFD3 -> ProteinMPNN -> Boltz-2 -> pDockQ2 ranking -> pH filter), ranking-function selection evidence, epitope selection |
| `02-key-results.md` | Submitted sets, same-target calibration (cetuximab Fab-EGFR), 7-definition hotspot study, cross-target pH finding |
| `03-limitations.md` | **Read this** — weak-proxy registry, statistical power, scope of claims |
| `metrics/` | Per-design metrics for all 72 designs (pDockQ2, ipSAE, ipTM, pTM, complex pLDDT, interface PAE, self-consistency, net charge, GRAVY, low complexity, longest homopolymer) |
| `structures/` | 86 files: 72 design models (Boltz-2) + 6 target/homology models + calibration controls |
| `05-methodology-addendum.md` | **Methodology addendum (M1-M4)**: epitope-definition study (7 definitions x 216 designs), C1 human/mouse cross-reactivity scoring with calibration, C3 agonist assessment (structure vs function), cross-target pH consistency |
| `reports/` | Full analysis reports: `c1-mouse-cross-report.md`, `c3-agonist-evidence.md` |
| `submissions/` | The three submitted sets (24 designs each) and unified 21-column traceability tables |

## Headline numbers

- C1 (EGFR): max `pDockQ2` **0.8003** — above the same-target real-antibody baseline
  (cetuximab Fab-EGFR, 0.4354, measured under identical conditions)
- C3 (GLP1R): `Jaccard` mean 0.335, 6 designs at Jaccard >= 0.5
- 72 designs total, 0 blocking validation failures

## Method summary

Generative backbone design (RFDdiffusion3) conditioned on experimentally derived
per-residue epitopes, sequence design (ProteinMPNN), structure re-prediction and
interface ranking (Boltz-2), with an orthogonal electrostatics-based pH-selectivity
filter for the two pH-switch challenges.

## Important

The designs and metrics in this package are **predictions** from computational
models. pDockQ2, pH ddE and Jaccard are used for **relative ranking only** and are
not claims of measured binding, affinity or function. See `03-limitations.md`
for the full weak-proxy registry and statistical caveats.

## Additional findings (addendum)

- **C1 cross-reactivity**: 10/24 designs pass the cross-reactivity criterion on the
  denominator basis, **6/24** when requiring same-index reproduction in all 3 diffusion
  samples. A cetuximab Fab control run under the identical protocol scores 0.9267 (human)
  vs 0.6081 (mouse) on the same epitope, so a human/mouse ratio of 0.42-0.50 alone is
  **not** evidence of human selectivity.
- **Epitope conservation**: the phrase "conserved epitope" is **not** used anywhere.
  Cross-species retention is governed by how well the *interface position set* is
  preserved, not by epitope conservation.
- **C3 agonist activity is NOT claimed**: 17 of 24 designs occupy the natural GLP-1
  position in an explicit ternary complex, and the construct excludes the transmembrane
  domain where class B GPCR activation occurs. These designs are presented as
  high-confidence binders with agonist activity registered as unknown.
- **Set-level footprint overlap**: the final C3 set shows Jaccard median 0.2936 /
  mean 0.3347 / coverage median 0.500 / max 0.833. The previously quoted 0.462 / 75%
  figure refers to a **single best design**, not the set.
