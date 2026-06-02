# Week 2 – Protein Quality Control & Aggregation
## Stochastic Gillespie Simulation with PQC Negative Feedback

**Course:** Physics of Molecular Diseases - Week 2
**Question of the week:** *"Where in the PQC network would you intervene to fight the aggregates?"*

---

## Background

When proteins misfold, the cell deploys **Protein Quality Control (PQC)** machinery — chaperones, proteases, and autophagy — to clear the damage. The key dynamical feature (from the lectures) is a **negative feedback loop**:

```
Aggregates  -->  upregulate PQC  --|  Aggregates
```

However, PQC has **finite capacity**. Once aggregates overwhelm it, a competing **positive feedback** takes over:

```
Aggregates  --|  PQC  --|  Monomers  -->  more Aggregates
```

This run-away positive feedback is the proposed mechanism behind the sharp, delayed onset of diseases like Alzheimer's and Parkinson's. The simulation here explores this transition stochastically using the **Gillespie SSA (Stochastic Simulation Algorithm)**.

---

## Reactions modelled

| # | Reaction | Propensity | Biology |
|---|----------|------------|---------|
| R1 | → M | `k_prod` | Constant production of unfolded/misfolded monomers |
| R2 | M → ∅ | `k_deg · M` | Basal monomer degradation |
| R3 | 2M → A | `k_nuc · M(M−1)/2` | Nucleation — Finke-Watzky slow step |
| R4 | M + A → 2A | `k_grow · M · A` | Autocatalytic elongation — Finke-Watzky fast step |
| R5 | A → M | `k_PQC_max · A / (K½ + A)` | **PQC clearance — negative feedback** |

**R5 is the key addition over the basic Finke-Watzky model.**  
Its Michaelis-like form captures two regimes:
- **Low A** → rate ≈ linear → PQC keeps up → healthy steady state
- **High A** → rate saturates at `k_PQC_max` → PQC overwhelmed → disease

---

## Parameter sets explored

| Scenario | `k_PQC_max` | `K½` | `k_prod` | Expected outcome |
|----------|-------------|------|----------|-----------------|
| Healthy (strong PQC) | 8.0 | 5.0 | 2.0 | Low ⟨A⟩ ≈ 0 |
| Borderline | 3.0 | 10.0 | 2.0 | Marginal — stochastic variability |
| Disease (weak PQC) | 1.0 | 10.0 | 2.0 | Run-away aggregation |
| Disease (high production) | 3.0 | 10.0 | **6.0** | Run-away aggregation (gene duplication model) |

All other base parameters: `k_deg = 0.04`, `k_nuc = 0.001`, `k_grow = 0.005`, `M0 = 50`, `A0 = 0`.  
Time is in arbitrary units (scale ~ proteasome half-life, ≈ 8–15 days in vivo).

---

## Figures produced

| Figure | Description |
|--------|-------------|
| **Fig 1** | M(t) and A(t) trajectories for all 4 scenarios — 8 stochastic runs + mean |
| **Fig 2** | PQC saturation curve (left) + histogram of steady-state A levels (right) |
| **Fig 3** | Single detailed trajectory — Healthy vs Disease side by side |
| **Fig 4** | Phase diagram: ⟨A⟩ at steady state as a heatmap over `k_PQC_max` × `k_grow` |

---

## Key results

```
Scenario                            <M>     <A>
─────────────────────────────────────────────────
Healthy (strong PQC)                35.0     0.6   ← low SS ✓
Borderline (moderate PQC)            1.1  1020.6
Disease (weak PQC)                   0.8  1076.3
Disease (high production)            0.8  3227.0
─────────────────────────────────────────────────
```

---

## Answering the week question: where to intervene?

The phase diagram (Fig 4) makes the answer quantitative. Four intervention points emerge from the model:

1. **Increase `k_PQC_max`** — boost PQC clearance capacity.  
   → *Biological analogue:* pharmacological activation of ABC transporters (e.g. thiethylperazine activating ABCC1, as shown in Krohn et al., *JCI* 2011). A mere **~11% increase in clearance** shifts the aggregation profile significantly (as the Krohn model predicts).

2. **Decrease `k_grow`** — inhibit autocatalytic elongation.  
   → *Biological analogue:* small molecules that cap fibril ends or disaggregate protofibrils.

3. **Decrease `k_prod`** — reduce the source of misfolded monomers.  
   → *Biological analogue:* siRNA knockdown of αSN or Aβ precursor; relevant because gene **duplication** of αSN alone is sufficient to cause familial Parkinson's (Chartier-Harlin et al. 2004).

4. **Reduce `K½`** — keep PQC sensitive at low aggregate levels, before saturation.  
   → *Biological analogue:* prevent the inhibition of PQC by insoluble aggregates (the `Ai --|PQC` arm in the Krohn model and the UPR/PERK pathway).

The most robust intervention from the phase diagram is **combining (1) and (3)** — increasing clearance *and* reducing production — because each alone shifts only one axis of the phase boundary.

---

## Connection to lecture material

| Concept | Where it appears in the code |
|---------|------------------------------|
| Finke-Watzky 2-step model | R3 (nucleation) + R4 (growth) |
| PQC negative feedback | R5 (Michaelis clearance) |
| Positive feedback / run-away | Emergent — when R4 outpaces R5 |
| Krohn et al. 2-module model | `k_PQC_max` ↔ transport capacity `c_T`; insoluble Agg ↔ `A` |
| Sneppen et al. Parkinson model | Proteasome saturation ↔ PQC saturation at high `A` |
| Hill function inhibition | `1/(1 + (Ai/K)^h)` from slides ↔ `A/(K½ + A)` here (h=1) |

---

## References

- Krohn et al. (2011). *Cerebral amyloid-β proteostasis is regulated by the membrane transport protein ABCC1 in mice.* Journal of Clinical Investigation, 121(10), 3924–3931.
- Sneppen et al. (2009). *Modeling proteasome dynamics in Parkinson's disease.* Physical Biology, 6, 036005.
- Finke & Watzky (1997). *A new mechanism for iridium(0) cluster growth.* JACS — the 2-step nucleation-growth model adapted for protein aggregation.
- Lecture slides: *Physics of Molecular Diseases – Week 2, Protein Quality Control* (Prof. Ala Trusina, NBI 2020).
