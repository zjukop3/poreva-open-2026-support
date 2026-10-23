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
