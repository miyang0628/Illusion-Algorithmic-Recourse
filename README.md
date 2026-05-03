# xai-inclusive-underwriting

> Replication repository for the paper submitted to an international peer-reviewed journal (under review).

---

## Overview

This repository contains the analysis code, governance evaluation scripts, and figure generation notebooks for a study on **explainable AI (XAI) and algorithmic fairness in health insurance underwriting**, using fasting plasma glucose (FPG) prediction as the primary application domain.

The study develops a four-stage audit framework that integrates:

1. **Stratified predictive modelling** across six age–sex demographic subgroups
2. **SHAP-based exclusion mechanism diagnosis**
3. **DiCE counterfactual accessibility analysis**
4. **A four-pillar AI governance scorecard** (Fairness, Robustness, Transparency, Accountability)

Key finding: Elderly Female applicants are correctly classified in only **43.8%** of cases vs **91.1%** for Young Female applicants (ΔStageAccuracy = 0.473), while Elderly Male applicants require **9.4×** the lifestyle modification burden of Young Male applicants to achieve the same favourable risk reclassification (mean Proximity: 0.974 vs 0.103).

---

## Repository Structure

```
xai-inclusive-underwriting/
│
├── notebooks/
│   ├── 01_regression_models.ipynb        # Stratified ML training & evaluation
│   ├── 02_governance_evaluation.ipynb    # Four-pillar governance scorecard
│   ├── 03_dice_counterfactual.ipynb      # DiCE counterfactual generation
│   └── 04_llm_report.ipynb              # GPT-4o-mini report generation + RAGAS
│
├── data/
│   └── README.md                         # Data access instructions (KNHANES)
│
├── figures/
│   ├── fig0_pipeline_overview.png        # Framework overview diagram
│   ├── fig1_fpg_distribution_by_group.png
│   ├── fig_g1_fairness.png
│   ├── fig_dice_proximity_comparison.png
│   ├── fig_governance_radar.png
│   ├── fig_ragas_summary.png
│   ├── fig_shap_summary_Young_Female.png
│   ├── fig_shap_summary_Elderly_Female.png
│   └── ...                               # Additional figures (see /figures)
│
├── tables/
│   ├── table1_descriptive_stats.csv
│   ├── table_g1_fairness.csv
│   ├── table_g2_robustness.csv
│   ├── table_g3_transparency.csv
│   ├── table_g4_accountability.csv
│   ├── table_governance_scorecard.csv
│   ├── table_ragas_summary.csv
│   └── dice_quality_metrics.csv
│
├── prompts/
│   └── fig0_pipeline_prompt_v2.json      # Image generation prompt for pipeline figure
│
├── requirements.txt
└── README.md
```

---

## Data

This study uses the **Korea National Health and Nutrition Examination Survey (KNHANES)**, a nationally representative, stratified, multi-stage probability sample conducted annually by the Korea Disease Control and Prevention Agency (KDCA).

- **Access**: Publicly available at [https://knhanes.kdca.go.kr](https://knhanes.kdca.go.kr)
- **Cycles used**: Three consecutive survey years (pooled)
- **Analytical sample**: n = 16,677 (after exclusions)
- **Raw data is not included in this repository** due to KDCA terms of use. Please download directly from the KNHANES portal and place files in the `/data` directory.

---

## Methods Summary

| Stage | Method | Tool |
|-------|--------|------|
| Predictive modelling | LR, Ridge, RF, LGBM, XGB, MLP | scikit-learn, LightGBM, XGBoost |
| Feature attribution | SHAP (TreeSHAP, KernelSHAP) | shap |
| Counterfactual explanation | DiCE | dice-ml |
| LLM report generation | GPT-4o-mini (temp=0.2) | openai |
| Report quality evaluation | RAGAS | ragas |
| Governance scoring | Custom four-pillar scorecard | — |

---

## Key Results

| Metric | Value |
|--------|-------|
| Best Stage Accuracy (Young Female) | 0.911 |
| Worst Stage Accuracy (Elderly Female) | 0.438 |
| ΔStageAccuracy | **0.473** (Risk threshold: 0.30) |
| Elderly Male Proximity (IFG→Normal) | **0.974** |
| Young Male Proximity (IFG→Normal) | 0.103 |
| Proximity ratio | **9.4×** |
| Governance composite score | **0.41 (Red — do not deploy)** |
| G1 Fairness | 0.05 ✗ Risk |
| G2 Robustness | 0.46 △ Caution |
| G3 Transparency | 1.00 ✓ OK |
| G4 Accountability | 0.38 △ Caution |
| LLM Context Precision | 1.000 |
| LLM Answer Relevancy | 0.818 |
| LLM Faithfulness | 0.251 |

---

## Requirements

```
python>=3.9
scikit-learn
lightgbm
xgboost
shap
dice-ml
openai
ragas
pandas
numpy
matplotlib
seaborn
jupyter
```

Install all dependencies:

```bash
pip install -r requirements.txt
```

---

## Reproducibility

All random seeds are fixed at `seed=42` throughout. Notebook execution order follows the numbering: `01` → `02` → `03` → `04`.

Note: Notebook `04_llm_report.ipynb` requires a valid OpenAI API key set as an environment variable:

```bash
export OPENAI_API_KEY=your_key_here
```

---

## Citation

If you use this code or data pipeline in your research, please cite:

```
[Anonymous authors]. (under review).
Algorithmic Exclusion in Health Insurance Underwriting:
An Explainable AI Audit Framework for Inclusive Insurance Governance.
Submitted to [Journal name anonymised for review].
```

This repository will be updated with full citation details upon acceptance.

---

## License

This repository is shared for research transparency and replication purposes.
Code is released under the [MIT License](LICENSE).
Data (KNHANES) is subject to KDCA terms of use — see [https://knhanes.kdca.go.kr](https://knhanes.kdca.go.kr).

---

## Disclosure

- **Conflicts of interest**: None declared.
- **Funding**: [Anonymised for review].
- **Ethics**: KNHANES data are de-identified and publicly available. No additional IRB approval was required.

---

*Repository maintained anonymously during peer review. Author details will be disclosed upon acceptance.*
