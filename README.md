# DWTS Fan Vote Reconstruction and Elimination System  
**MCM 2026 · Problem C**

This repository presents a complete, end-to-end modeling pipeline for **Problem C (Dancing With the Stars)** of the 2026 MCM/ICM.  
The central challenge is that **fan votes are unobserved**, while judge scores and eliminations are observed.  
We reconstruct latent fan preferences, validate existing voting rules, explain fan behavior, and design a new elimination system that balances **fairness, accuracy, and competitive excitement**.

---

## 1. Problem Framing

In each week \( $t$ \), contestants receive:
- a judge score \( $J_{i,t}$ \),
- an unobserved fan vote share \( $F_{i,t}$ \),
- and typically one contestant is eliminated.

The key difficulty is that **fan votes are never directly observed**.  
Elimination outcomes only provide *relative information*:  
the eliminated contestant must have performed worse than the remaining contestants under some combined judge–fan mechanism.

This motivates a **latent-utility modeling approach**.

---

## 2. Latent Fan Model (Problem 1)

We introduce a latent fan utility

$\eta_{i,t} = a_{i,s} + \beta_J \tilde{J}_{i,t} + \beta_t t,$

where:
- \( $a_{i,s}$ \) is a season-specific baseline popularity,
- \( $\tilde{J}_{i,t}$ \) is the standardized judge score,
- \( $t$ \) captures time effects.

Fan vote shares are defined via a softmax transformation:

$$F_{i,t} = \frac{e^{\eta_{i,t}}}{\sum_{j \in \mathcal{A}_t} e^{\eta_{j,t}}}.$$

Eliminations impose inequality constraints:  
the eliminated contestant must have lower effective support than all others.  
We estimate \( $\eta_{i,t}$ \) by maximizing a likelihood derived from these constraints.

**Output:** `fan_shares_estimated.csv`

---

## 3. Rule Validation and Comparison (Problem 2)

Using reconstructed fan shares, we evaluate common elimination rules:

- **Rank Sum Rule:** sum of judge rank and fan rank,  
- **Percent Sum Rule:** sum of normalized judge and fan percentages,  
- **Bottom-Two + Judge (BT+J):** judges decide between the bottom two by percent.

We measure:
- disagreement rates between rules,
- how often each rule matches historical eliminations,
- judge–fan conflict frequency.

This reframes voting rules as **mechanisms that can be empirically tested**, rather than assumed.

---

## 4. Explaining Fan Preferences (Problem 3)

To interpret fan behavior, we decompose latent fan utility:

$$
\eta_{i,t} = \underbrace{\eta^{\text{lin}}_{i,t}}_{\text{performance + time}} + \underbrace{\eta^{\text{ml}}_{i}}_{\text{attributes}}
$$


- A **linear backbone** explains judge performance and temporal trends.
- A **GBDT residual model** captures nonlinear effects of:
  age, industry, professional partner, and background.

This decomposition yields interpretable insights such as baseline popularity by industry or professional partner.

**Output:** `panel_with_eta_decomp.csv`

---

## 5. Designing a New Elimination System (Problem 4)

We propose **CHER** (Composite Hybrid Elimination Rule), a dynamic elimination system:

$S_{i,t} = α_J(t) J_{i,t} + α_F(t) F_{i,t} + α_M M_{i,t}.$

where:
- \( $\alpha_J(t)$ \) increases over the season,
- \( $\alpha_F(t)$ \) decreases accordingly,
- \( $M_{i,t}$ \) captures fan momentum.

### Elimination Procedure
1. Select the bottom two contestants by \( $S_{i,t}$ \) (with fairness shields).
2. Eliminate the contestant with lower fan support within the bottom two.

CHER achieves:
- high agreement with historical eliminations,
- reduced judge–fan conflict,
- controlled fan inequality.

---

## 6. Repository Structure

```text
.
├── Latex
├────── reference.bib
├────── easymcm.sty
├────── New_Template.tex
├──────Graph
├── Problem_Codes_including_plots
├────── Problem1-Estimate_fans.ipynb
├────── Problem2-Verify_Data_and_System.ipynb
├────── Problem3-Fully_Analysis.ipynb
├────── Problem4-Design_new_System.ipynb
├── fan_shares_estimated.csv
├── 2026_MCM_Problem_C_Data.csv
├── 2026_MCM_Problem_C.pdf
└── README.md
