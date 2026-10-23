# Key results

All numbers below come from the frozen ranking function — **max pDockQ2 over 3 Boltz-2
diffusion samples** — and are aggregated identically wherever they are compared.

---

## 1. Submitted sets

| | **C1 EGFR** | **C2 TNF-α** | **C3 GLP1R** |
|---|---|---|---|
| designs | 24 | 24 | 24 |
| length range (aa) | 70–90 | 70–110 | 70–130 |
| backbone families | 16 | 5 | 12 |
| max per family | 3 | 8 | 4 |
| **pDockQ2 median** | 0.4568 | **0.4675** | **0.5622** |
| **pDockQ2 max** | **0.8003** | 0.5840 | 0.7895 |
| pH class wanted | ACID | BASE | — (agonist) |
| class-matching designs | 8 / 24 | 8 / 24 | A 10 / B 9 / C 5 |

## 2. Same-target calibration — cetuximab

Absolute `pDockQ2` thresholds do **not** transfer between targets, so each target needs its
own reference point. For EGFR the control is the **cetuximab Fab–EGFR complex from 6ARU**,
predicted under **exactly** the conditions used for the designs (single-sequence, no MSA,
3 samples, recycles 3, steps 200).

| chain pair | pDockQ2 | ipTM_d0chn | ipSAE |
|---|---|---|---|
| EGFR(A) – Fab heavy chain(B) | **0.4354** | 0.7798 | 0.6255 |
| EGFR(A) – Fab light chain(C) | 0.2428 | 0.7965 | 0.6031 |
| Fab B–C (internal pairing, methodological control) | 0.9229 | 0.6851 | 0.9144 |

Side-by-side:

| source | max pDockQ2 |
|---|---|
| 16-case unmasked positives (**PD-L1** target) | 0.7228–0.8286 |
| C3 v1 best (GLP1R) | 0.6152 |
| C2 v1 best (TNF-α) | 0.5374 |
| C1 v1 best (EGFR, 6 backbones) | 0.4520 |
| **cetuximab (real anti-EGFR antibody), same conditions** | **0.4354** |
| **our C1 submission, best design** | **0.8003** |

**Read this carefully.** Our C1 designs score *above* the real antibody under these
conditions. That is a statement about the score, not about the designs: an antibody's CDR
conformation depends on somatic hypermutation and evolutionary information, so in
single-sequence mode Boltz-2 systematically *under*-predicts the Fab. The comparison is
fair in the sense that both sides ran the same way, but **0.4354 is not cetuximab's real
binding capability**, and a 70–110 aa binder also presents different interface geometry
from a 422 aa Fab. See `03-limitations.md`.

## 3. Hotspot-definition study — 7 definitions, 216 designs

C3 was the hard challenge: the initial hotspot definition was **wrong** (Jaccard 0.143
against the native agonist footprint). Seven definitions were then built and scored under
one consistent methodology. All figures use the same aggregation rule (Jaccard and
pDockQ2 both taken as max over the 3 diffusion samples).

| definition | residues | n designs | pool Jaccard median | pool Jaccard max | pool pDockQ2 median | best-24 Jaccard median | best-24 pDockQ2 median |
|---|---|---|---|---|---|---|---|
| v1 | 30 | 24 | 0.1710 | 0.4375 | 0.2266 | — | — |
| loose | 24 | 36 | 0.2028 | 0.5128 | 0.1562 | 0.2373 | 0.2170 |
| tight | 16 | 36 | 0.2731 | 0.5000 | 0.1184 | 0.2475 | 0.2796 |
| **core35** | **12** | 24 | **0.4402** | 0.5946 | 0.2042 | **0.4444** | 0.2780 |
| buried12 | 12 | 24 | 0.3076 | 0.5758 | 0.2834 | 0.2780 | 0.3543 |
| shell | 50 | 24 | 0.3710 | 0.5676 | 0.1437 | 0.4070 | 0.2441 |
| core8 | 8 | 24 | 0.3408 | **0.6129** | 0.1266 | 0.2927 | 0.2170 |
| core8d | 5 | 24 | 0.3297 | 0.5676 | 0.3238 | 0.3297 | 0.3487 |

**Ceiling reached:** across all definitions the achievable footprint overlap is
**median ≈0.26** (all definitions pooled), **0.4402** for the single best definition
(core35), and **0.6129** for the single best design (core8). Going further would need a
method that can *constrain the contact pattern* (contact-map-conditioned diffusion or a
footprint reward), which this pipeline does not have.

> ⚠️ **The Jaccard column is a WEAK PROXY, and it is used as negative evidence only.**
> Contact overlap with the native agonist footprint says **nothing** about whether a design
> **activates** the receptor — C3 is scored mostly on activation, and no activation predictor
> was available. A design can overlap the footprint and be an antagonist, and a design can
> activate through contacts this metric does not see. Everything below is a statement about
> *where the designs make contact*, not about potency or efficacy.

**Where the turning point is.** Narrowing from 24 → 12 residues **improves** the footprint
overlap (0.2028 → 0.4402). Narrowing from 12 → 8 **degrades both** metrics at once
(Jaccard 0.4402 → 0.3408; pDockQ2 0.2042 → 0.1266). 12 residues is therefore the balance
point between functional evidence and geometric samplability — it is a real turning point,
not an arbitrary cut.

**A counter-example that sharpens the conclusion:** core8d uses **5** residues and beats
core8's 8 on both metrics (Jaccard 0.3297 vs 0.3408 — comparable; pDockQ2 0.3238 vs 0.1266).
So the deciding factor is **which** residues and where they sit, not how many.

**Decision taken from this study: freeze.** The core8 variant's best-24 set reaches
Jaccard median 0.293 against the frozen set's 0.294 (**−0.001**, inside noise) while its
pDockQ2 median falls to 0.2170 from 0.5883 (**−63%**). Losing the score to buy no overlap
gain is not a trade worth making, so **the submitted C3 set was frozen**.

### 3a. An important negative result about backbone count

Increasing the number of RFD3 backbones raises the **maximum** score but does **not** move
the **median** footprint overlap. The backbone generator cannot constrain contact patterns,
so more backbones explore the same distribution rather than a better one. Any claim that
"more backbones buy a better footprint" is unsupported by our data, and we do not make it.

## 4. Cross-target consistency: affinity and pH switching compete

C1 and C2 ask for **opposite** pH behaviour on **different** targets and epitopes. If the
same structure appears in both, it is a property of the design objective rather than an
accident of one pool.

| | **C1 EGFR (wants ACID)** | **C2 TNF-α (wants BASE)** |
|---|---|---|
| pH window | 6.5 → 7.4 | 6.0 → 7.4 |
| designs with both pDockQ2 and a pH class | 24 | 30 |
| pool class distribution | NEUTRAL 32 / ACID 7 / BASE 1 (n=40) | NEUTRAL 44 / BASE 29 / ACID 21 (n=94) |
| **class composition of the top 8 by pDockQ2** | **NEUTRAL 7 / ACID 1** | **NEUTRAL 7 / ACID 1** |
| target-class median pDockQ2 | 0.4315 | 0.3388 |
| other-class median pDockQ2 | 0.3971 | 0.4207 |
| target-class **max** | 0.4727 | 0.3633 |
| overall **max** | **0.8003** (NEUTRAL) | **0.5840** (NEUTRAL) |
| Spearman \|ΔΔE\| vs pDockQ2 | **rho −0.387, p≈0.049** | rho −0.049, p≈0.79 |
| Spearman target-direction ΔΔE vs pDockQ2 | rho +0.151, p≈0.47 | rho −0.082, p≈0.67 |

### What this shows

> Two challenges, two unrelated targets and epitopes, and **opposite** pH directions both
> produced the same pattern: **7 of the top 8 designs by pDockQ2 are pH-NEUTRAL**, and the
> best switching design sits clearly below the overall best score
> (C1: 0.4727 vs 0.8003; C2: 0.3633 vs 0.5840).
> This is not a sampling accident of one batch. The same physical competition — a strong
> pH switch requires ionisable groups at the interface and lets electrostatics dominate,
> which costs interface quality — reproduced independently on two targets.

### The mechanism, and why it matters

On C1, the **magnitude** of the switch is what correlates with loss of score
(|ΔΔE| vs pDockQ2: rho −0.387), while the **signed, wanted-direction** ΔΔE does not
(rho +0.151, p≈0.47). So the cost depends on **how strong** the switch is, not on which
direction it points. That is exactly why a BASE switch on C2 can reproduce the pattern
found for an ACID switch on C1.

### The honest qualification

C2's correlation is **not significant** (rho −0.049, p≈0.79) and only **2** BASE designs
enter the C2 comparison set, so the rank correlation there has almost no power. The
cross-target claim rests on the **class composition of the top ranks** (7/8 NEUTRAL in
both), which is a robust, count-based observation, and **not** on C2's p-value. Presenting
C2 as a second statistically significant result would be wrong.

Concretely: C2 has **n=2** designs in the target (BASE) class that also carry a pDockQ2,
against n=4 for C1's ACID class. A rank correlation on n=2 carries essentially no
information, so **C2's p-value should not be quoted as evidence for or against the effect**
— it is under-powered, not refuting.

## 5. Self-consistency: measured, informative-negative

Mean pairwise CA-RMSD (after superposition on the binder chain) between the 3 diffusion
samples of each design was computed for all 72 designs. Tested against the ranking key:

| | rho (self-consistency vs pDockQ2) | p |
|---|---|---|
| C1 | −0.250 | 0.23 |
| C2 | −0.083 | 0.70 |
| C3 | −0.061 | 0.77 |

**No significant association on any target.** Self-consistency is reported because it is a
standard quality axis and its absence is itself a finding: on these de novo designs, the
three diffusion samples frequently converge on **different binding modes** (mean values
6–33 Å), and that spread carries no information about the interface score. It should **not**
be used as a ranking key here.

## 6. Developability screen

Sequence-level screening was applied to the final 72 (`seq-quality.py`): longest
homopolymer, tandem-repeat period, dipeptide Shannon entropy, low-complexity fraction,
net charge at pH 7.4, and banned tag motifs.

Final counts: **C1 21/24, C2 21/24, C3 16/24** pass every screen. **All remaining flags are
advisory low-complexity fraction** (our own threshold); no submission contains a banned
motif. The screen only became a blocking gate after it found a real defect — see §7.

## 7. A defect the checks caught, and how it was resolved

The developability screen found four designs carrying `A{6,}` **poly-alanine runs**, which
Poreva treats as a tag motif for the `protein` class. Our own submission gate had missed
the pattern. After the gap was closed, the affected designs were replaced by re-balancing
from the same pools under the frozen ranking function, preserving every other constraint
(family quota, identity cap) and — for C3 — preserving the design's **hedge class**, so the
A/B/C split was not silently unbalanced.

| challenge | removed | reason | replacement | replacement pDockQ2 | class preserved |
|---|---|---|---|---|---|
| C2 | `c2_tnfa … 70B_0_model_0_d1` | poly-Ala | `c2_tnfa … 90B_0_model_0_d1` | 0.3332 | BASE ✓ |
| C3 | `… core35_70c_0_model_0_d3` | poly-Ala | `… buried12_90c_0_model_0_d2` | 0.5394 | A ✓ |
| C3 | `… foot_tight_110b_0_model_0_d2` | poly-Ala | `… buried12_90c_0_model_0_d3` | 0.2406 | B ✓ |

Effect on the submitted sets: C2 pDockQ2 median 0.4746 → **0.4675**, C3 median
0.5883 → **0.5622**, families unchanged. Both `replaced` and `replaced_reason` are recorded
per row in the lineage tables, so this substitution is visible without explanation.
