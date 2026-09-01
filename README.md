# The Illusion of Algorithmic Recourse

> Replication repository for a manuscript submitted to an anonymous peer-reviewed journal (under review).

**How fairness and explainability metrics conceal structural exclusion in AI-driven health insurance underwriting.**

---

## Overview

This repository contains the analysis code and figure/table generation notebooks for a study on how **explainable-AI (XAI) governance and recourse metrics** behave when applied to health-insurance underwriting, using fasting plasma glucose (FPG) prediction on the Korea National Health and Nutrition Examination Survey (KNHANES) as the empirical setting.

The central finding is methodological: the seemingly precise numbers produced by an XAI audit (a fairness score, a "9.4x recourse burden") are **highly sensitive to three hidden normative choices** —

1. **which fairness metric** is used (base-rate-confounded vs base-rate-invariant),
2. **what is held immutable** in a counterfactual (structural variables such as income and education),
3. whether **feature interdependence and small-sample uncertainty** are reported.

When these choices are made explicit, a "measured unfairness" either **dissolves into a metric artefact** or is **redefined as a more fundamental exclusion**: for one subgroup, favourable recourse does not merely cost more — it **does not exist**.

---

## Key results

| Finding | Evidence |
| --- | --- |
| Raw stage-accuracy gap is base-rate confounded | ΔStageAccuracy = **0.463** vs ΔBalancedAccuracy = **0.096**; most groups do not beat a majority-class baseline |
| Deployment verdict flips with the fairness metric | Composite = **0.41 (Red)** under raw G1 vs **0.63 (Amber)** under base-rate-corrected G1 |
| The real deployment blocker is robustness, not "unfairness" | Temporal validation degrades in every group (e.g. RMSE 7.8 → 20.6); R² ≈ 0.02–0.16 |
| The "9.4x burden" does not reproduce | With structural variables immutable, Elderly Female / Young Male proximity ratio = **1.12x** (Euclidean), **1.92x** (Mahalanobis) |
| **Structural recourse absence** (headline) | Elderly Male IFG→Normal: **0 / 324** eligible cases reach Normal across **5,000** dense actionable samples each; median minimum achievable prediction = **103.4 mg/dL** (a wall above the Normal threshold) |

The recourse-absence finding is triangulated by **three independent methods**: the group's confusion matrix (zero Normal predictions), DiCE counterfactual search (0/324), and a direct decision-boundary probe (0/324).

---

## Repository structure

```
.
├── data/                       # KNHANES SAS files (not included — see below)
│   ├── hn20_all.sas7bdat
│   ├── hn21_all.sas7bdat
│   ├── hn22_all.sas7bdat
│   ├── hn23_all.sas7bdat
│   └── hn24_all.sas7bdat
├── notebooks/
│   ├── 01_regression_models.ipynb            # stratified prediction + base-rate diagnostics
│   ├── 02_governance_evaluation.ipynb        # four-pillar scorecard (G1–G4), dual reporting
│   ├── 03_dice_stage1_diagnostic.ipynb       # DiCE recourse diagnostic (actionability, Mahalanobis, bootstrap CI)
│   └── 03b_dice_stage2_recourse_absence.ipynb# decision-boundary probe (recourse-absence confirmation)
└── results/
    ├── figures/                # generated figures (grayscale, PNG + PDF, 600 dpi)
    ├── tables/                 # generated CSV tables
    └── artifacts/              # intermediate objects passed between notebooks (.pkl, preprocessed data)
```

---

## Data

This study uses the **Korea National Health and Nutrition Examination Survey (KNHANES)**, a nationally representative, stratified, multi-stage probability sample conducted annually by the Korea Disease Control and Prevention Agency (KDCA).

- **Access**: publicly available at <https://knhanes.kdca.go.kr>.
- **Cycles used**: five consecutive cycles (hn20–hn24, 2020–2024), pooled.
- **Raw data is not included** in this repository under the KDCA terms of use. Download the five `*.sas7bdat` files from the KNHANES portal and place them in `data/`.
- **All five cycles are required.** The diabetes-medication flag (`HE_DMdr`) is present only in the 2020–2021 cycles, so the medication exclusion runs correctly only when all five files are loaded.

### Sample construction (CONSORT)

| Step | n before | n after | removed |
| --- | ---: | ---: | ---: |
| Raw merged (hn20–hn24) | — | 34,640 | — |
| 1. Drop missing FPG | 34,640 | 30,392 | 4,248 |
| 2. Exclude diabetes-medication users | 30,392 | 30,344 | 48 |
| 3. Restrict to adults (age ≥ 19) | 30,344 | 27,936 | 2,408 |
| 4. FPG within 40–400 mg/dL | 27,936 | **27,934** | 2 |

---

## Method summary

| Stage | Method | Tools |
| --- | --- | --- |
| Stratified prediction | LR, Ridge, RF, LightGBM, XGBoost, MLP (Optuna-tuned), 6 age–sex strata | scikit-learn, LightGBM, XGBoost, Optuna |
| Fairness diagnostics | Majority baseline, balanced accuracy, Cohen's κ, macro-F1, confusion matrices | scikit-learn |
| Governance scorecard | Four pillars (G1 Fairness, G2 Robustness, G3 Transparency, G4 Accountability), dual G1 reporting, weight sensitivity | custom + SHAP |
| Recourse diagnostic | DiCE (random), actionable-only, Euclidean + Mahalanobis proximity, bootstrap CIs | dice-ml |
| Recourse-absence probe | Dense actionable-space sampling of the model's decision boundary | NumPy (vectorised) |

Structural variables (income quartile, household income, education, health-screening) are held **immutable** in all recourse analyses; only clinically modifiable lifestyle and diet variables may vary.

---

## Reproducibility

- All random seeds are fixed at `seed = 42`.
- Notebooks are **path-robust**: each locates the project root by finding the folder that contains both `data/` and `results/`, so they run whether launched from the repository root or from `notebooks/`.
- Recommended execution order: `01` → `02` → `03` → `03b`.
- Notebook `02` and `03b` load only the primitive artefacts saved by `01` (fitted models, preprocessed data, configuration); they recompute their own diagnostics and are otherwise self-contained.

### Requirements

```
python>=3.9
scikit-learn
lightgbm
xgboost
optuna
shap
dice-ml
pyreadstat
pandas
numpy
matplotlib
seaborn
joblib
```

Notes:
- `pyreadstat` is required to read the KNHANES `*.sas7bdat` files.
- SHAP values for tree models are computed via each library's **native** contribution path (`pred_contribs` / `pred_contrib`) rather than the SHAP library's tree parser, to avoid a known incompatibility between recent XGBoost and SHAP versions.
- Figures are written as grayscale PNG **and** PDF at 600 dpi.

---

## Outputs

Running the notebooks regenerates, under `results/`:

- **Tables**: CONSORT flow, descriptive statistics with base rates, performance diagnostics, governance scorecard, recourse availability and proximity tables, bootstrap confidence intervals.
- **Figures**: base-rate vs corrected fairness, governance radar, RMSE heatmap, recourse-availability and decision-boundary-probe plots.

---

## License

- Code is released under the [MIT License](LICENSE).
- KNHANES data are subject to KDCA terms of use — see <https://knhanes.kdca.go.kr>.

---

## Disclosure

- **Conflicts of interest**: none declared.
- **Ethics**: KNHANES data are de-identified and publicly available; no additional IRB approval was required.

---

*Repository maintained anonymously during peer review. Author and full citation details will be added upon acceptance.*
