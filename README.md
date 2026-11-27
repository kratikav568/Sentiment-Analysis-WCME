# Working Capital Management, Firm Performance & BRSR Disclosure: Evidence from Nifty 500 Firms

> Does the sustainability-reporting regime introduced by SEBI's **Business Responsibility and Sustainability Report (BRSR)** change how a firm's **cash conversion cycle (CCC)** relates to its **profitability and market value**? A fixed-effects panel study of Indian listed firms, 2019–2026.

![Python](https://img.shields.io/badge/Python-3.9%2B-blue)
![pandas](https://img.shields.io/badge/pandas-data%20wrangling-150458)
![SciPy](https://img.shields.io/badge/SciPy-statistics-8CAAE6)
![Status](https://img.shields.io/badge/status-complete-brightgreen)

---

## Table of Contents
1. [Overview](#overview)
2. [Research Questions & Hypotheses](#research-questions--hypotheses)
3. [Data](#data)
4. [Variables](#variables)
5. [Methodology](#methodology)
6. [Key Findings](#key-findings)
7. [Selected Figures](#selected-figures)
8. [Repository Structure](#repository-structure)
9. [How to Run](#how-to-run)
10. [Limitations & Future Work](#limitations--future-work)
11. [Author](#author)

---

## Overview

Every firm decides how long to wait for customers to pay, how much inventory to hold and how long to stretch its own suppliers. Together these choices set the **cash conversion cycle (CCC)**. Most emerging-market studies find that longer cycles go with weaker returns, but almost none ask whether the *information environment* (how much and how positively a firm reports on sustainability) changes that relationship. India is also thinly covered, and no prior study places a before/after design around a reporting reform.

This project builds an eight-year panel of **Nifty 500 firms (2019–2026)** and tests whether BRSR **disclosure intensity** and **narrative sentiment** moderate the link between working capital (lnCCC) and performance (**ROA, ROE, Tobin's Q**).

**Headline result:** the pooled relationship is statistically zero, but this hides two groups of firms with opposite slopes and an **inverted-U** shape for ROA and ROE. BRSR disclosure and tone show **no robust moderating effect**.

| | |
|---|---|
| **Period** | 2019–2026 (pre-mandate 2019–22, post-mandate 2023–26) |
| **Full sample (Sample A)** | 370 non-financial firms, 2,764 firm-years |
| **BRSR sample (Sample B)** | 330 firms, 1,087 firm-years (2023–26) |
| **Estimator** | OLS with firm and year fixed effects, firm-clustered standard errors |
| **Tools** | Python, pandas, NumPy, SciPy, Matplotlib |

---

## Research Questions & Hypotheses

| ID | Hypothesis |
|---|---|
| **H1** | A longer cash conversion cycle lowers firm performance. |
| **H2** | The cycle–performance relationship is an **inverted U** with an interior optimum. |
| **H3** | BRSR **disclosure intensity** (ESGDI) buffers the effect of the cycle on performance. |
| **H4** | BRSR **narrative sentiment** (ESG-NSI) moderates the relationship independently of disclosure volume. |
| **H5** | BRSR shifts the **optimal** cycle. |
| **H7** | The relationship changes after the mandate, more so for high-disclosure firms. |
| **H8** | Higher-BRSR firms close the gap to a peer-target cycle faster. |
| **COVID test** | Firms with longer cycles entering 2020 lost more performance. |

---

## Data

Two Excel workbooks, both in wide format, reshaped to long firm-year panels in the notebook.

| File | Content | Coverage |
|---|---|---|
| `Final_Data_Minor_project.xlsx` | Financial and governance variables, plus India CPI inflation and GDP growth | 499 Nifty 500 firms × 2019–2026 (3,992 firm-years) |
| `BRSR_Scores.xlsx` | Text-derived BRSR scores (ESGDI, ESG-NSI, overall and net sentiment, E/S/G pillar shares) | 488 firms × 2023–2026 (1,607 firm-years) |

**Sample construction**

| Stage | Firms | Firm-years |
|---|---|---|
| Financial file (all firms) | 499 | 3,992 |
| Firms with at least one valid lnCCC | 370 | 2,960 |
| **Sample A**: firm-years with lnCCC and ROA | 370 | 2,764 |
| **Sample B**: Sample A, 2023–26, with BRSR scores | 330 | 1,087 |

Key data decisions (all documented in the notebook):
- Company names differ between files, so they were standardised and matched exactly, then fuzzy-matched at a 0.85 similarity cut-off (462 of 488 BRSR firms matched).
- 129 firms (26%) with no valid lnCCC in any year, mainly banks, NBFCs and insurers, were dropped because the cycle is not meaningful for them.
- Impossible values were set to missing (negative firm age, negative log assets, tangibility above 1, boards of fewer than 3).
- 2026 leverage was carried forward from 2025. Continuous variables were winsorised at the 1st and 99th percentiles.
- BRSR scores are **not** created or back-filled for years before 2023.

> **Data source note:** *(add the provider of the financial data here, for example the database or terminal used, and confirm redistribution is permitted before publishing the raw files.)*

---

## Variables

| Variable | Definition | Role |
|---|---|---|
| **lnCCC** | Natural log of the cash conversion cycle (DIO + DSO − DPO), centred at its sample mean | Working-capital measure |
| **ROA** | Net profit / total assets × 100 | Performance |
| **ROE** | Net profit / shareholders' equity × 100 | Performance |
| **Tobin's Q** | (Market value of equity + total debt) / total assets | Performance |
| **ESGDI** | Share of scored BRSR sentences that are ESG-related | Moderator (disclosure intensity) |
| **ESG-NSI** | Mean positive minus negative probability over ESG sentences | Moderator (narrative sentiment) |
| Net / overall sentiment, E / S / G shares | Alternative tone measures and pillar decomposition | Sensitivity |
| Firm size, leverage, tangibility | Log total assets, debt / assets, fixed assets / total assets | Controls |
| Promoter holding, board independence, board size | Governance controls | Controls |
| R&D intensity + reporting dummy | R&D / sales × 100; 1 if R&D reported above zero | Controls |
| Inflation, GDP growth, COVID dummy | Macro controls (absorbed by year effects in FE models) | Controls |

ESGDI and ESG-NSI are standardised (mean 0, SD 1), so each interaction coefficient is the change in the lnCCC slope for a one-SD higher score.

> The BRSR text scores are provided in `BRSR_Scores.xlsx`. The notebook consumes them as inputs and does not run the NLP scoring itself.

---

## Methodology

All models use **firm fixed effects** (within transformation), usually with **year fixed effects**, and **standard errors clustered by firm**. The estimator is implemented directly in NumPy/pandas (`fe_ols` in the notebook), with a small-sample cluster-robust variance correction. Pooled OLS is reported alongside for comparison.

| Step | Model / test | Sample |
|---|---|---|
| Baseline (H1) | `Perf = α + β1·lnCCC + γ'X + μ_i + λ_t + ε` | A |
| Optimum (H2) | Adds `lnCCC²`; inverted U needs β2 < 0, turning point inside the observed range, positive slope at P5 and negative slope at P95 | A |
| Moderation (H3, H4) | `lnCCC × BRSR` interaction; ESGDI and ESG-NSI separately, together, as abnormal tone, and by E / S / G pillar | B |
| Shifted optimum (H5) | BRSR interacted with both lnCCC and lnCCC² | B |
| Adjustment speed (H8) | Change in within-year percentile rank regressed on the gap to the size-and-year peer median, and Gap × BRSR | B |
| Pre/post (H7) | `lnCCC × Post`, then `lnCCC × Post × HighBRSR`, plus an event-study version (2022 as reference year) | A |
| COVID shock | 2019 lnCCC (and rank) × COVID-year dummy | A |
| Endogeneity checks | Lagged lnCCC → performance, lagged ROA → lnCCC | A |
| Multiple testing | **Holm-adjusted p-values** across the six primary moderation tests | B |
| Diagnostics | Variance inflation factors (all between 1.03 and 1.30) | A |

Because every Nifty 500 firm is under the mandate, there is no untreated group, so all results are interpreted as **associations, not causal effects**.

---

## Key Findings

| Hypothesis | Result | Key evidence |
|---|---|---|
| H1: Longer CCC lowers performance | **Not supported** on the full sample; **supported** for long-cycle firms (lnCCC > 0) | FE coefficient near zero (p = 0.35–0.96); upper-cluster ROA slope −0.75 (p = 0.003) |
| H2: Inverted U with an optimum | **Supported** for ROA and ROE; not for Tobin's Q | Squared term −0.047 for ROA (p < 0.001); turning point about lnCCC −0.8 to −1.1 |
| H3: Disclosure intensity moderates | **Not supported** in the pooled sample; exploratory signal in long-cycle firms | Pooled p = 0.60–0.84; long-cycle ROA interaction +0.69 (p = 0.035, Holm p = 0.21) |
| H4: Tone moderates beyond disclosure | **Not supported** | All p > 0.17 |
| H5: BRSR shifts the optimum | **Not supported** | Interaction with squared term p = 0.12–0.14 |
| H7: Link changes after the mandate | **Mixed / not supported** | ROE and Tobin's Q shift in opposite directions; triple interaction p ≥ 0.14 |
| H8: High-BRSR firms adjust faster | **Not supported** | Interaction p = 0.59–0.71 |
| COVID test | **Suggestive for Tobin's Q** | 2019 rank × COVID = −0.95 (p = 0.015); no effect on ROA or ROE |

**Main takeaways**
1. **lnCCC is bimodal** (clusters near −4.5 and +4, about 33% of firm-years above zero). Pooling the two clusters produces a null average slope that masks opposite effects: negative for long-cycle firms, positive for short-cycle firms.
2. **A curved relationship fits better than a linear one.** Performance peaks at an intermediate cycle length and falls at both extremes. The turning point sits in a sparsely populated region between the two clusters, so it is best read as "very short-cycle firms do well, long-cycle firms do badly."
3. **BRSR disclosure and tone do not robustly moderate the relationship.** BRSR scores vary little across firms (SD about 0.04–0.06) and about two-thirds of that variation is between firms, which limits what a fixed-effects interaction can detect.
4. **Long-cycle firms with more disclosure lose less ROA** (about −2.3 points per lnCCC unit at the 10th percentile of ESGDI versus about −0.55 at the 90th), but this does not survive multiple-testing correction and rests on 97 firms. It is a lead, not a finding.
5. **No sign of reverse causality** in lagged-variable checks (all p > 0.7), although with a null baseline this is not conclusive.

---

## Selected Figures

**Distribution of lnCCC (bimodal)**

![lnCCC distribution](images/fig1_lnccc_distribution.png)

**Yearly trends in lnCCC, ROA, ROE and Tobin's Q (dotted line marks the BRSR mandate)**

![Yearly trends](images/fig2_yearly_trends.png)

**Inverted-U fit of ROA on lnCCC (5th–95th percentile)**

![Quadratic fit](images/fig6_quadratic_fit.png)

**Slope of lnCCC at low, median and high BRSR scores (95% CI): no moderation in the pooled sample**

![BRSR moderation](images/fig7_brsr_moderation.png)

**Event study: high-minus-low BRSR slope differences (reference year 2022)**

![Event study](images/fig8_event_study.png)

**Long-cycle firms (lnCCC > 0): effect of lnCCC on ROA by BRSR level**

![Long-cycle firms](images/fig9_long_cycle_firms.png)

<details>
<summary>More figures</summary>

**ROA across lnCCC deciles**

![ROA deciles](images/fig4_roa_by_decile.png)

**Correlation matrix (Sample A)**

![Correlation matrix](images/fig5_correlation_matrix.png)

**ESGDI and ESG-NSI distribution by year, 2023–2026**

![BRSR box plots](images/fig3_brsr_boxplots.png)

</details>

---

## Repository Structure

```
.
├── README.md
├── WCM_BRSR_Analysis.ipynb          # Full analysis notebook (cleaning → models → robustness)
├── Final_Data_Minor_project.xlsx    # Financial, governance and macro data (499 firms, 2019–2026)
├── BRSR_Scores.xlsx                 # BRSR text-derived scores (488 firms, 2023–2026)
├── MinorProject_FinalReport.pdf     # Full written report
└── images/                          # Figures used in this README
```

The notebook reads both Excel files from its own folder, so keep them side by side (or update the two `pd.read_excel(...)` paths in the notebook if you move them).

### Notebook outline
1. Load libraries, then load and reshape both workbooks (wide to long)
2. Merge on standardised company names (exact plus fuzzy matching)
3. Data-quality checks: missingness, impossible values, shape of lnCCC
4. Cleaning, transformation (centring, standardising, winsorising) and sample construction
5. Exploratory analysis: descriptives, pre/post comparison, trends, BRSR distributions, correlations, VIF, selection check
6. Regressions: baseline, non-linearity, pre/post, BRSR moderation, optimum shift, adjustment speed, triple interaction and event study, COVID test
7. Endogeneity checks, robustness grid, long-cycle subsample analysis and hypothesis scorecard

---

## How to Run

```bash
# 1. Clone the repository
git clone https://github.com/<kratikav568>/<Sentiment-Analysis-WCME>.git
cd <Sentiment-Analysis-WCME>

# 2. Install dependencies
pip install pandas numpy scipy matplotlib openpyxl skimpy jupyter

# 3. Launch the notebook and run all cells
jupyter notebook WCM_BRSR_Analysis.ipynb
```

**Libraries used:** `pandas`, `numpy`, `scipy`, `matplotlib`, `skimpy` (summary tables), `openpyxl` (Excel reader), plus the standard-library `re` and `difflib`. The regression code needs no `statsmodels`.
---

## Limitations & Future Work

**Limitations**
- Only lnCCC was supplied, and a log is undefined for zero or negative cycles, which removed 129 firms. Results apply to firms with a computable positive cycle.
- Only four BRSR years exist, so within-firm variation is limited and fixed-effects interactions have low power.
- Firms with BRSR scores are larger and more profitable than those without, which narrows generalisation.
- Industry codes are not in the data, so instrumental-variable and system-GMM estimates were not feasible.
- Inventory, receivables and payables days were not available, so component-level models could not be run.
- The BRSR tone measure reflects how firms write, and the sentiment model was not validated against hand-coded sentences.
- 2026 leverage is carried forward from 2025.
- All results are associations because the mandate covers every firm.

**Future work**
- Rebuild the cycle from raw inventory, receivables and payables days using a transformation that preserves negative values, which would bring the excluded firms back.
- Add component-level (DIO / DSO / DPO) models, industry controls and financial-constraint proxies.
- Validate sentiment scores against hand-coded text, and extend the post-mandate window as more BRSR years become available.

---

## Author

**\<Kratika\>**
\<MSc Business Analytics / Manipal Academy of Higher Education\> · (https://www.linkedin.com/in/kratika568/) ·

---

## Citation

If you use this work, please cite it as:

> \<Kartika\> (2026). *Sentiment Analysis of Working Capital Management Efficiency on the Firm Performance of Nifty 500 Keeping BRSR as the Moderator, Pre & Post 4 Years.* GitHub repository.
